# Agent Harness의 구조와 Deterministic Control Loop

> Sources: Tejas Kumar·IBM, YouTube (Unknown); Jesse Vincent interview, YouTube (Unknown)
> Raw: [Harnesses in AI transcript](../../raw/software-career/harnesses-in-ai-a-deep-dive-tejas-kumar-ibm.md); [Original YouTube source provenance](../../raw/software-career/harnesses-in-ai-tejas-kumar-source-provenance.md); [Fixing AI Slop interview transcript](../../raw/software-career/fixing-ai-slop-manage-agents-like-mit-interns.md)
> Updated: 2026-08-16

## Overview

Agent harness는 model에게 더 긴 prompt를 주는 기술이 아니라 **non-deterministic model 주변에 안정적인 실행 환경과 검증 loop를 두는 system**이다. Model이 tool을 선택하고 작업을 수행하더라도, harness는 사용 가능한 tool, context, 실행 횟수, 성공 판정, retry, trace와 secret 경계를 제어한다. 핵심은 agent의 자기 보고를 믿지 않고 관찰 가능한 상태로 완료 여부를 다시 판정하는 것이다.

## Harness는 Model 주변의 전체 System이다

| 구성요소 | 역할 |
|---|---|
| Model | 다음 행동과 tool 호출을 선택 |
| Tool registry | 읽기·쓰기·shell·browser 등 허용된 행동 정의 |
| Context management | 시스템 지침, task, history와 compaction 관리 |
| Guardrail | 실행 step, message, 비용과 금지 행동의 경계 설정 |
| Agent loop | Model 응답과 tool 결과를 반복 연결 |
| Trace | 실제 tool 호출, 상태 변화와 실패 기록 |
| Verify | 외부에서 성공 조건을 재판정 |
| Retry·escalation | 실패 후 재시도 상한과 사람에게 넘길 조건 정의 |

Claude Code, Cursor와 Codex처럼 coding agent도 model만이 아니라 file·shell tool, context 관리와 실행 loop를 포함하므로 harnessed agent로 볼 수 있다. 특정 product를 harness라고 부르는지보다 이 구성요소가 실제로 어떻게 작동하는지가 중요하다.

## Prompt를 바꾸지 않고 결과를 바꾸는 과정

발표의 browser agent는 특정 행동을 수행하라는 task를 받았다. 초기 agent는 login page로 이동한 뒤 실제 성공하지 못했지만 성공했다고 보고했다. 발표자는 prompt를 강화하지 않고 harness를 점진적으로 보강했다.

### 실행 상한과 Context 경계

Agent가 무한히 tool을 호출하거나 history가 계속 커지지 않도록 실행 step과 message 상한을 둔다. Context가 커지면 중요한 초기 지침과 최근 상태를 남기고 중간 history를 정리한다. 발표의 압축 방식은 교육용으로 단순화된 예시이며 production 최적화 방식은 아니다.

### Trace를 기반으로 성공을 다시 판정

Agent의 최종 문장이 아니라 browser URL, tool 호출, login 처리 여부와 화면 상태를 살펴보는 deterministic verifier를 추가한다. 이 단계에서 구현은 아직 성공하지 못했지만, 적어도 agent의 거짓 완료 선언을 실제 실패로 바꾸었다.

```text
Agent: 완료했음
Harness: Trace와 실제 상태 검사
└── 조건 미충족 → 실패
```

정확한 실패를 만드는 것이 성공을 만드는 첫 단계다. 실패를 성공으로 오판하면 retry나 개선이 시작될 수 없다.

### 안정적인 절차를 Deterministic Code로 옮김

Login 상태를 감지하면 harness가 직접 관리된 경로로 credential을 주입하고 form을 제출하도록 했다. Agent에게 credential을 prompt로 넘겨 매번 login 방법을 추론하게 하는 대신, 확실히 알고 있는 절차는 일반 code가 담당한다.

Secret을 code에 하드코딩하라는 뜻은 아니다. Harness는 environment variable이나 credential store처럼 관리된 경로에서 secret을 읽고 model context에 노출하지 않도록 접근 경계를 제한해야 한다.

### Retry는 상한이 있는 Control Loop다

```text
Attempt
→ Agent loop
→ Trace
→ Deterministic verify
   ├─ 성공 → 종료
   ├─ 실패 + 재시도 가능 → 다음 Attempt
   └─ 실패 + 상한 도달 → Human escalation
```

같은 상태에서 무한히 반복하는 것은 reliability가 아니라 비용과 시간의 낭비다.

## Coding Agent에 대응하면

| Browser Agent | Coding Agent |
|---|---|
| Browser tool | File·shell·test tool |
| URL·click trace | Command·diff·test result |
| Login handler | Project setup·generated code·migration 전용 script |
| Behavior verifier | Test·typecheck·lint·build·behavior check |
| Attempt 상한 | Repair loop 상한과 escalation |

Agent에게 “Test를 실행해”라고 말하는 것은 workflow instruction이다. Agent loop 밖의 harness가 직접 `verify` command를 실행하고 exit code와 diff를 검사한 후 실패 시 종료를 거부하는 것이 deterministic control이다.

## 주니어용 Minimum Harness

1. Agent가 사용할 file·shell tool과 금지 범위를 정한다.
2. 실행한 command, 변경 file과 test 결과를 trace로 남긴다.
3. Repository의 단일 `verify` command를 만든다.
4. Agent의 완료 선언 후에 harness가 `verify`를 직접 실행한다.
5. 검증 실패를 구조화해 agent에게 되돌린다.
6. Retry 상한 후에는 사람에게 증거와 함께 넘긴다.
7. Test 삭제·비활성화와 검증 범위 축소는 별도 diff rule과 human review로 확인한다.

Superpowers처럼 brainstorming·plan·TDD·review 순서를 제공하는 Skill은 workflow layer다. 이 위에 repository `verify`, completion gate, trace, retry 상한과 escalation을 두면 자연어 지침이 실제 control loop로 보강된다.

## Behavioral Proof와 Skill 공급망

Jesse Vincent의 Superpowers 인터뷰는 “완료했다”는 문장 대신 실제 product를 사용하는 screenshot·video tour 같은 proof를 요구하는 사례를 제시한다. Coding harness에서는 unit test만이 아니라 필요한 경우 browser·API·CLI의 사용자 흐름을 실행하고, 그 trace와 결과물을 verifier가 확인하도록 확장할 수 있다. 인터뷰 시점의 Superpowers에는 독립된 behavioral testing 단계가 완전히 내장된 것은 아니므로, workflow Skill과 end-to-end verifier를 같은 것으로 간주하지 않는다.

Skill은 model context에 들어가는 단순 참고 문서가 아니라 사용자 권한으로 tool 실행을 유도할 수 있다. 따라서 출처, 변경 내역, 자동 update 여부와 permission 범위를 audit한다. Workflow Skill이 오염되더라도 위험 command gate, secret boundary와 completion verifier가 독립적으로 남아 있어야 한다.

## 한계

발표의 harness는 개념을 보여주기 위한 최소 browser demo다. Context 압축, credential 관리, verifier의 충분성과 retry 정책을 production에 그대로 사용해야 한다는 근거가 아니다. Harness가 성공으로 판정하는 조건 자체가 부족하면 잘못된 결과를 안정적으로 반복할 수 있으므로, 사용자 outcome과 검증 자산의 무결성은 사람이 검토한다.

## See Also

- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md)
- [AI Coding Autonomy Experiment와 Human-in-the-loop](ai-coding-autonomy-experiment.md)
- [Brownfield AI Agent Workflow](brownfield-ai-agent-workflow.md)
- [Claude Code Hooks](claude-code-hooks.md)
- [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](superpowers-agent-management-and-spec-driven-development.md)
