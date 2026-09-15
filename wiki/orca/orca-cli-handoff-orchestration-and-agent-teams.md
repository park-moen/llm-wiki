# Orca CLI Handoff, Orchestration과 Claude Agent Teams

> Sources: 사용자 현업 회고, 2026-08-19~2026-08-20; Stably, Inc., Unknown; Anthropic, Unknown
> Raw: [4레인 병렬 개발 Field Note](../../raw/orca/2026-08-19-orchestration-vs-agent-teams-field-note.md); [Orca Orchestration](../../raw/orca/cli-orchestration.md); [Claude Code에서 Agent 병렬 실행](../../raw/ai-agents/claude-code-run-agents-in-parallel.md)
> Updated: 2026-08-20

## Overview

Orca Worktree에 agent를 실행했다고 해서 자동으로 Orca orchestration이 되는 것은 아니다. Orca CLI handoff는 별도 작업 공간과 process에 일을 넘기지만 완료 계약을 만들지 않는다. Orca supervised orchestration은 Run·Task·Dispatch와 `worker_done`을 연결해 coordinator가 결과를 추적하게 한다. Claude Code Agent Teams는 lead, shared task list와 inter-agent messaging을 이용해 Claude 내부에서 팀을 조정하지만 teammate를 자동으로 Git Worktree에 격리하지는 않는다.

## 현장 기록에서 실제로 사용한 방식

Jira `TNSP-1233` 아래 하위 작업 8건을 처리하기 위해 4개의 Orca 레인을 만들었다. 구현 레인은 다음 형태의 명령으로 시작했다.

```text
orca worktree create --agent claude --prompt ...
```

이 명령은 Worktree를 만들고 Claude에게 작업을 넘기는 데는 성공했지만, orchestration Run·Task·Dispatch를 만들지는 않았다. 중간 지시도 `orca terminal send`로 직접 전달했다. 따라서 실제 구조는 supervised orchestration이 아니라 **Orca CLI Worktree handoff**였다.

완료된 agent의 terminal output, Git commit, PR과 `lastOutputAt`을 별도로 읽어야 했던 이유도 여기에 있다. Worker에게 `worker_done`을 보낼 task ID와 dispatch ID가 주어지지 않았으므로 coordinator에게 완료 event가 도착할 수 없었다.

## 세 가지 조정 모델

| 방식 | 조정 주체 | 완료와 결과 | 파일 격리 |
|---|---|---|---|
| Orca CLI handoff | 사용자 또는 외부 coordinator | 자동 완료 계약 없음 | 새 Orca Worktree를 만들면 격리 가능 |
| Orca supervised orchestration | Orca Run에 연결된 coordinator | Dispatch의 `worker_done`, question과 escalation | 현재 Worktree 또는 새 child·top-level Worktree 선택 |
| Claude Code Agent Teams | Lead Claude | Shared task list와 inter-agent messaging | Teammate 자동 Worktree 격리 없음 |

Claude Code의 `Agent` tool subagent와 Agent Teams도 구분해야 한다. 현장 기록의 조사 단계에서는 7개의 `Agent` tool subagent를 사용했고 `<task-notification>`으로 결과를 받았다. 이는 lead와 teammate가 공유 task list를 운영하는 Agent Teams를 실제로 실행한 사례가 아니다.

## Polling이 필요했던 이유와 대안

Handoff에는 coordinator lifecycle이 없으므로 terminal과 Git 상태를 polling하는 것이 당시의 완료 판별 방법이었다. Polling 과정에서 commit convention 차이, 자율적인 PR 생성과 레인 간 디자인 token 차이를 완료 전에 발견한 것은 실제 이점이었다.

그러나 중간 관찰과 완료 추적을 polling 하나로 처리할 필요는 없다. Supervised orchestration에서는 다음 역할을 분리할 수 있다.

- `worker_done`: 성공 또는 실패로 작업이 끝났음을 한 번 보고한다.
- `heartbeat`: 장시간 작업이 살아 있음을 알린다.
- `ask`: worker가 blocking 결정을 coordinator에게 묻는다.
- `check --wait`: coordinator가 완료·질문·escalation event를 기다린다.
- `worker-read`: 필요할 때 worker output을 제한된 범위로 확인한다.

`check --wait`는 고정 간격으로 모든 Worktree를 순회하는 loop가 아니라 message가 도착할 때까지 기다리는 event 기반 대기다. 다만 Run은 durable namespace와 inbox이지 background scheduler가 아니다. Coordinator agent가 종료되어 있으면 message는 남지만, model이 저절로 다시 실행되는 것은 아니다.

## Agent Teams로 바꾸면 해결되는 것과 남는 것

Agent Teams를 사용하면 lead가 shared task list와 teammate message를 통해 완료와 중간 결정을 관리하므로, 독립 Worktree를 사람이 순회하는 부담은 줄어든다. 특히 서로 결과를 주고받아야 하는 조사·review·가설 검증에 자연스럽다.

반면 Agent Teams만으로 다음 문제까지 자동 해결되지는 않는다.

- 같은 파일을 수정하는 teammate의 작업 디렉터리와 branch 격리
- Orca Worktree별 dev server, browser와 사용자의 시각적 검수
- Lead session 밖에서 만들어진 Orca worker의 lifecycle 추적
- 사람이 검토할 수 있는 범위를 넘어선 병렬 변경의 통합

따라서 Agent Teams는 coordination을, Worktree는 filesystem과 branch isolation을 해결한다. 둘은 대체 관계가 아니다.

## 다음 작업의 선택 기준

- 작업 전체를 다른 Worktree에 넘기고 사용자가 직접 관찰한다면 Orca CLI handoff를 사용한다.
- Main workspace가 완료를 기다리고 결과에 따라 후속 Task를 실행해야 한다면 처음부터 Run·Task·Dispatch를 가진 Orca supervised orchestration을 사용한다.
- 짧은 조사 결과만 parent context로 회수한다면 Claude subagent가 단순하다.
- Claude teammate끼리 shared task와 message가 필요하다면 Agent Teams를 사용하되 파일 소유권을 분리한다.
- UI 작업처럼 사람이 화면을 확인해야 한다면 completion event와 별도로 human checkpoint를 둔다.

## 운영 교훈

도구 이름이나 Worktree의 branch 이름으로 orchestration 사용 여부를 판단하지 않는다. 실제로 다음 객체가 존재하는지 확인한다.

```text
Run
└── Task
    └── Dispatch
        └── Worker
            └── worker_done
```

이 연결이 없다면 agent가 아무리 여러 Orca Worktree에서 병렬 실행되더라도 그것은 lifecycle이 추적되는 orchestration이 아니라 독립 실행 또는 handoff다.

## See Also

- [Orca CLI·자동화·오케스트레이션](orca-cli-automation-and-orchestration.md)
- [AI Agent Teams와 Git Worktree](../ai-agents/agent-teams-and-git-worktrees.md)
- [Orca 핵심 모델과 첫 멀티에이전트 세션](orca-foundations-and-worktree-model.md)
