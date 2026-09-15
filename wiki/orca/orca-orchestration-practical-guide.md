# 이전 4레인 작업을 Orca Orchestration으로 실행하는 방법

> Sources: [Orca CLI Handoff, Orchestration과 Claude Agent Teams](orca-cli-handoff-orchestration-and-agent-teams.md); [Orca CLI·자동화·오케스트레이션](orca-cli-automation-and-orchestration.md); [AI Agent Teams와 Git Worktree](../ai-agents/agent-teams-and-git-worktrees.md)
> Archived: 2026-08-20

## Overview

이 문서는 Jira `TNSP-1233`의 하위 작업 8건을 4개의 Orca Worktree에서 병렬로 처리했던 경험을 바탕으로, 완료 계약이 없는 Orca CLI handoff를 실제 supervised orchestration으로 전환하는 방법을 정리한 시점 고정 가이드다. 핵심은 Worktree를 만드는 것보다 먼저 Run과 Task를 만들고, worker를 반드시 Dispatch에 연결하는 것이다.

```text
이전
worktree create --prompt
→ 독립 Handoff
→ 완료 계약 없음
→ /loop로 polling

앞으로
Run 생성
→ Task 생성
→ Worker를 Dispatch에 연결
→ worker_done 수신
→ 결과 검토와 후속 작업
```

## 이번 작업에 맞는 전체 구조

이전 `TNSP-1233` 작업을 orchestration으로 다시 구성한다면 다음처럼 나눌 수 있다.

```text
Main Orca Workspace
└── Run: TNSP-1233 사용자 화면 피드백 반영
    ├── Task A: TNSP-1236
    │   └── develop 기반 Worktree
    ├── Task B: TNSP-1237
    │   └── develop 기반 Worktree
    ├── Task C: TNSP-1238 → TNSP-1242 데모1
    │   └── feat/demo1 기반 Worktree
    ├── Task D: TNSP-1239 → 1240 → 1241 → 1242 데모2
    │   └── feat/demo2 기반 Worktree
    └── Task E: TNSP-1243 통합 검수
        └── Task A~D 완료 후 시작
```

Task C와 D 내부의 작업은 같은 파일과 상태에 의존하므로 기존처럼 한 Worktree 안에서 순차 실행한다.

## 사전 조건 확인

Orca에서 **Settings → Experimental → Orchestration**을 활성화한다. 명령 형태가 Orca 버전에 따라 바뀔 수 있으므로 현재 설치된 runtime의 가이드를 먼저 읽는다.

```bash
orca skills get orchestration --full
```

확인: `Preferred Supervised Worker Loop`, `worker-start`와 `worker_done` 설명이 표시되어야 한다.

Orca runtime 연결 상태를 확인한다.

```bash
orca status --json
```

확인: `ok: true`와 `runtime.state: "ready"`가 나와야 한다.

## Main Workspace에서 Run 생성

Main workspace의 coordinator terminal에서 전체 목표를 등록한다.

```bash
orca orchestration run-create --objective "TNSP-1233 사용자 화면 피드백 하위 작업을 Worktree별로 구현하고 검수한다" --json
```

확인: 새 `run_id`와 현재 coordinator 연결 정보가 반환되어야 한다.

Run은 Task와 message를 모으는 durable namespace다. Worker를 자동으로 만들거나 완료된 coordinator model을 다시 실행하는 background scheduler는 아니다.

## 독립 Task 생성

모든 독립 Task를 먼저 만든 뒤 worker를 병렬로 시작한다.

```bash
orca orchestration task-create --spec "TNSP-1236 메인 페이지 Hero를 수정한다. develop을 기준으로 작업하고 검증 후 커밋한다. push와 PR은 사용자 확인 전 실행하지 않는다." --json
```

확인: 반환된 `task_id`를 Task A로 기록한다.

```bash
orca orchestration task-create --spec "TNSP-1237 상단 메뉴 진입 경로를 수정한다. develop을 기준으로 작업하고 검증 후 커밋한다. push와 PR은 사용자 확인 전 실행하지 않는다." --json
```

확인: 반환된 `task_id`를 Task B로 기록한다.

```bash
orca orchestration task-create --spec "feat/demo1을 기준으로 TNSP-1238을 끝낸 뒤 TNSP-1242의 데모1 범위를 순차 구현한다. 각 티켓을 별도 커밋하고 사용자 확인 전 PR을 만들지 않는다." --json
```

확인: 반환된 `task_id`를 Task C로 기록한다.

```bash
orca orchestration task-create --spec "feat/demo2를 기준으로 TNSP-1239, TNSP-1240, TNSP-1241, TNSP-1242의 데모2 범위를 이 순서로 구현한다. 각 티켓을 별도 커밋하고 사용자 확인 전 PR을 만들지 않는다." --json
```

확인: 반환된 `task_id`를 Task D로 기록한다.

## 일반적인 Worker 시작 방법

Repository 기본 base branch에서 시작해도 되는 Task는 `worker-start`를 사용하는 것이 가장 단순하다.

```bash
orca orchestration worker-start --task <task_a_id> --worktree new-top-level --repo name:fe --name tnsp-1236-main-hero --agent claude --setup run --json
```

확인: `dispatch_id`, agent terminal handle, 생성된 Worktree와 branch를 확인한다.

`worker-start`는 Worktree 생성, agent terminal 실행, lifecycle preamble 주입과 Dispatch 연결을 하나의 흐름으로 구성한다. 이전의 `worktree create --prompt`와 달리 worker는 자신이 어떤 Task와 Dispatch에 속하는지 알게 된다.

## 서로 다른 Base Branch가 필요한 경우

이번 사례처럼 `develop`, `feat/demo1`, `feat/demo2`로 base가 다르면 Worktree를 먼저 정확히 만들고 Task를 `dispatch --inject`로 연결할 수 있다.

이때 Worktree 생성 명령에는 작업 prompt를 넣지 않는다. Prompt 대신 Dispatch가 Task와 lifecycle 계약을 전달해야 한다.

```bash
orca worktree create --repo name:fe --name tnsp-1238-demo1-hero --base-branch feat/demo1 --agent claude --setup run --json
```

확인: `result.worktree.branch`와 `agentTerminalHandle` 또는 `startupTerminal.handle`을 확인한다.

이전 field note에서는 `--base-branch feat/demo1`을 사용했는데 기존 branch가 그대로 checkout된 사례가 있었다. 작업을 시작하기 전에 실제 branch가 의도한 작업 branch인지 반드시 확인한다.

Claude terminal이 입력을 받을 준비가 됐는지 기다린다.

```bash
orca terminal wait --terminal <agent_terminal_handle> --for tui-idle --timeout-ms 60000 --json
```

확인: terminal이 `tui-idle` 상태에 도달해야 한다.

Task를 terminal에 연결한다.

```bash
orca orchestration dispatch --task <task_c_id> --to <agent_terminal_handle> --inject --json
```

확인: `dispatch_id`가 반환되고 lifecycle preamble이 성공적으로 주입되어야 한다.

`--inject`가 worker에게 Task spec, `task_id`, `dispatch_id`, 질문 방식과 `worker_done` 보고 계약을 전달한다. Worktree를 만들었더라도 이 연결이 없으면 supervised orchestration이 아니다.

## 실제 Orchestration인지 확인

Worker를 시작한 뒤 Task와 Dispatch가 존재하는지 확인한다.

```bash
orca orchestration task-list --brief --json
```

확인: 시작한 Task의 상태가 `dispatched`여야 한다.

```bash
orca orchestration dispatch-show --task <task_id> --json
```

확인: 연결된 worker terminal과 active Dispatch가 표시되어야 한다.

다음 lifecycle 기록이 없다면 orchestration이 아니라 독립 실행 또는 handoff다.

```text
Run
└── Task
    └── Dispatch
        └── Worker
            └── worker_done
```

## 시간 기반 Loop 대신 완료 Event 기다리기

모든 독립 worker를 시작한 뒤 coordinator는 다음 event를 기다린다.

```bash
orca orchestration check --wait --types worker_done,escalation,question --timeout-ms 900000 --json
```

확인:

- `worker_done`: 성공 또는 실패로 작업 종료
- `question`: worker가 사용자 결정을 기다림
- `escalation`: blocker 또는 coordinator 개입 필요

이는 고정 간격으로 모든 terminal을 읽는 polling이 아니다. Message가 도착하면 반환하는 event 기반 대기다. Timeout은 worker 실패를 의미하지 않는다. Worker가 계속 작업 중이면 rolling wait를 다시 실행한다.

## Worker의 질문에 답하기

Worker가 PR 생성 여부나 공통 component 수정 범위를 물었다면 coordinator가 답한다.

```bash
orca orchestration reply --id <question_message_id> --body "현재 Worktree 범위만 수정하고, 공통 component 변경은 별도 Task로 남겨라." --json
```

확인: Reply가 원래 question thread에 연결되어야 한다.

사용자가 Orca 앱에서 worker terminal에 직접 답했다면 Main coordinator에도 그 사실을 알려야 한다. 그렇지 않으면 지시 채널이 둘로 나뉘어 모순된 명령이나 중복 작업이 생길 수 있다.

## 완료 결과와 Human Checkpoint

`worker_done`을 받았다고 바로 완료를 승인하지 않는다. 먼저 worker 결과를 읽는다.

```bash
orca orchestration worker-read --dispatch <dispatch_id> --limit 100 --json
```

확인:

- 변경 파일
- 실행한 검증과 생략한 검증
- Commit 여부
- 남은 위험과 사용자 확인 사항

UI 작업은 별도의 human checkpoint를 둔다.

```text
worker_done
→ Diff 확인
→ dev server 실행
→ Orca Browser에서 화면 확인
→ 필요하면 follow-up Task
→ 승인 후 push·PR
```

`worker_done`은 worker가 맡은 작업을 끝냈다는 보고이지 사용자가 결과를 승인했다는 뜻이 아니다.

## 완료된 Worker 정리

같은 worker에게 바로 후속 Task를 맡기지 않는다면 release한다.

```bash
orca orchestration worker-release --dispatch <dispatch_id> --json
```

확인: Worker output이 보존되고 orchestration이 소유한 agent terminal만 정리되어야 한다.

Message를 모두 처리하고 worker의 reuse·retain·release를 결정한 다음 Delivery를 acknowledge하고 다시 기다린다.

```bash
orca orchestration check --ack <delivery_id> --wait --types worker_done,escalation,question --timeout-ms 900000 --json
```

확인: 이전 Delivery가 처리되고 다음 event 대기로 진입해야 한다.

## 통합 검수 Task 실행

Task A~D가 모두 완료된 뒤 `TNSP-1243` 검수 Task가 준비되도록 dependency를 지정한다.

```bash
orca orchestration task-create --spec "TNSP-1243 통합 브라우저·기기 검수를 수행하고 발견된 문제를 보고한다. 자동 수정하지 말고 사용자 승인 gate에서 멈춘다." --deps '["<task_a_id>","<task_b_id>","<task_c_id>","<task_d_id>"]' --json
```

확인: 선행 Task가 끝나기 전에는 `pending`, 모두 끝난 뒤에는 `ready`가 되어야 한다.

준비된 Task를 확인한다.

```bash
orca orchestration task-list --ready --json
```

확인: Task A~D가 완료된 뒤 검수 Task가 나타나야 한다.

## Claude Code Agent Teams와 결합하기

처음부터 모든 Worktree에서 Agent Teams를 사용하지 않는다. 먼저 다음 구조를 안정화한다.

```text
Main Orca coordinator
└── Orca Task
    └── Claude worker 1개
        └── worker_done
```

특정 Task가 충분히 복잡할 때만 Claude worker가 내부에서 Agent Teams를 사용하게 한다.

```text
Main Orca coordinator
└── Orca Task C
    └── Claude lead
        ├── 조사 teammate
        ├── 구현 teammate
        └── 리뷰 teammate
```

책임 경계는 다음과 같다.

- Orca는 Claude lead 하나를 Worker로 추적한다.
- Agent Teams 내부 Task는 Claude lead가 관리한다.
- Teammate별 수정 파일을 분리한다.
- 최종 검증과 결과 종합은 Claude lead가 담당한다.
- Orca `worker_done`은 Claude lead가 한 번만 전송한다.
- 사용자는 Main workspace에서 top-level Task와 human checkpoint를 관리한다.

Agent Teams teammate는 자동으로 Worktree에 격리되지 않는다. 같은 파일을 동시에 수정하는 작업에는 명확한 파일 소유권이나 별도 격리 전략이 필요하다.

## 이전 작업과 달라져야 할 체크리스트

### Worker 시작 전

- [ ] Run과 Task를 먼저 만들었는가?
- [ ] Full handoff인가, supervised 작업인가?
- [ ] Base branch와 실제 작업 branch를 확인했는가?
- [ ] Commit·push·PR 중 어디에서 멈출지 적었는가?
- [ ] `.env` 전달 방식과 dev server port를 정했는가?
- [ ] UI 확인을 human checkpoint로 두었는가?

### Worker 시작 후

- [ ] Task 상태가 `dispatched`인가?
- [ ] Active Dispatch가 존재하는가?
- [ ] Worker가 lifecycle preamble을 받았는가?
- [ ] Main coordinator가 `check --wait`하고 있는가?
- [ ] 사용자의 직접 개입을 coordinator에게 공유했는가?

### 완료 후

- [ ] `worker_done`의 outcome이 명시됐는가?
- [ ] 실제 test·build·diff를 확인했는가?
- [ ] 다음 Task로 재사용할지 release할지 결정했는가?
- [ ] Delivery를 처리한 뒤 acknowledge했는가?

## 최종 운영 원칙

Main workspace가 작업 결과를 이어받아야 한다면 `worktree create --prompt`로 넘기지 않는다. Run과 Task를 먼저 만들고 `worker-start` 또는 `dispatch --inject`로 lifecycle을 연결한다.

Agent Teams는 Orca orchestration의 대체재가 아니다. 복잡한 Orca Task 내부에서 Claude lead가 사용할 수 있는 선택적 실행 계층이다.

## See Also

- [Orca CLI Handoff, Orchestration과 Claude Agent Teams](orca-cli-handoff-orchestration-and-agent-teams.md)
- [Orca CLI·자동화·오케스트레이션](orca-cli-automation-and-orchestration.md)
- [AI Agent Teams와 Git Worktree](../ai-agents/agent-teams-and-git-worktrees.md)
