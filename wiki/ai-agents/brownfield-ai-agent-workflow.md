# Brownfield AI Agent Workflow

> Sources: Personal brownfield harness design, 2026-08-10; Personal Superpowers Practice, 2026-08-11
> Raw: [brownfield 하네스](../../raw/ai-agents/2026-08-10-brownfield-harness.md); [Superpowers Issue 0 실행 기록](../../raw/ai-agents/2026-08-11-superpowers-issue-0-run.md)
> Updated: 2026-08-11

## Overview

기획 문서가 부족한 기존 service를 AI agent로 수정할 때는 전체 codebase를 먼저 문서화하지 않는다. 실행 중인 code를 source of truth로 보고, 요청과 연결된 blast radius만 탐색하며, 현재 동작을 characterization test로 고정한 다음 변경한다. 작업하면서 실제로 알게 된 지식만 후불로 적립한다.

## 성격과 현재 상태

이 문서는 인수인계받은 brownfield project에 적용할 **개인 workflow 설계 제안**이다. 원본 작성 시점에는 실제 hook과 skill이 구현·검증되지 않았으므로 검증된 product 사용법으로 취급하지 않는다. 방법론을 시범 적용해 telemetry와 실패 사례가 쌓이면 상태와 규칙을 다시 갱신해야 한다.

> **Status: Outdated** (2026-08-11)
> 원본 작성 시점의 “실사용 검증 없음” 상태는 안동 project의 Superpowers Issue 0 품질 baseline 실행으로 일부 바뀌었다. 착륙·계획·실행·review·local branch 병합과 병합 후 verify까지 한 차례 관찰했지만, 이후 기능 Issue와 remote CI는 아직 검증하지 않았다.

## 발상의 전환: 정본은 실행 중인 code

Greenfield workflow는 PRD와 specification을 source of truth로 삼기 쉽다. Brownfield project에서는 이미 배포되어 동작하는 code가 현재 시스템의 실질적인 정본이다. 과제는 전체 specification을 새로 쓰는 것이 아니라, 변경과 관련된 현재 동작을 실행 가능한 test로 추출하는 것이다.

Characterization test는 올바르다고 기대하는 동작이 아니라 **변경 전 code가 실제로 하는 동작**을 기록한다. 현재 동작이 bug처럼 보여도 먼저 그대로 고정하고, 의도적으로 바꾸는 단계에서 test 변경을 기록한다.

## 개인 원칙

| 원칙 | 의미 |
|---|---|
| 정본은 code | planning document가 없다고 멈추지 않고 실행 가능한 현재 동작에서 시작 |
| 범위는 blast radius | 전체 codebase 이해 대신 요청과 연결된 실행 경로만 탐색 |
| 안전망 먼저 | 변경 전에 characterization test로 현재 동작을 고정 |
| 문서는 후불 적립 | 작업 중 실제로 확인한 영역만 기록 |

가장 중요한 범위 제약은 codebase 전체를 먼저 이해하려 하지 않는 것이다. 요청 하나의 entry point에서 시작해 실제 호출 경로, 공유 code와 회귀 후보까지만 조사한다.

## 5-Stage Workflow

| Stage | 빈도 | 목적 | Gate |
|---|---|---|---|
| 착륙 | repository당 처음 | 실행과 test baseline을 green으로 만듦 | test command 성공 |
| 조준 | 요청마다 | 관측 가능한 동작과 blast radius 확정 | 범위가 크면 분할 권고 |
| 박제 | 요청마다 | 현재 동작을 characterization test로 고정 | 변경 전 code에서 test 통과 |
| 변경 | task마다 | RED, GREEN, REFACTOR로 요구사항 구현 | 회귀 판정, 전체 test, test-presence |
| 적립 | 요청마다 | 새로 알게 된 내용만 문서화 | 별도 차단 없음 |

## Stage 0: 착륙

먼저 service 실행 방법과 test command를 확인한다. Test suite가 이미 green이면 baseline으로 채택한다. 기존 test가 red라면 수정하거나 명시적인 skip 사유를 기록해 baseline을 구분할 수 있게 만든다. Test가 전혀 없으면 framework와 smoke test를 최소한으로 부트스트랩한다.

Baseline이 red인 상태에서 변경하면 기존 실패와 새 회귀를 구분할 수 없으므로 이후 gate의 의미가 사라진다.

첫 산출물은 `LANDING.md` 같은 작은 landing note다. 복사 가능한 build·run·test command, 주요 directory의 책임, 외부 의존, 현재 위험 지대만 기록한다.

## Stage 1: 조준

사용자 요청을 관측 가능한 동작 하나로 줄이고 entry point에서 실행 경로를 따라간다. 변경 대상뿐 아니라 같은 code를 공유하는 다른 동작도 회귀 후보로 기록한다.

```text
사용자 요청
└── entry point
    └── service 또는 core function
        ├── 실제 변경 지점
        └── 공유 dependency
            └── 회귀 후보
```

Blast radius가 너무 크면 그대로 진행하지 않고 요청을 더 작은 동작으로 분리한다.

## Stage 2: 박제

로컬 실행이 가능하면 API 같은 outside-in 수준에서 현재 요청과 응답을 먼저 고정한다. 이후 실제로 변경할 function이나 class에만 좁은 단위 test를 추가한다.

Characterization test가 진짜 현재 동작을 기록하는지는 변경 내용을 잠시 제외한 상태에서 test가 통과하는지 확인해 기계적으로 판정할 수 있다.

```bash
git stash push --include-untracked
<characterization test command>
git stash pop
```

복원 실패는 작업 data에 영향을 줄 수 있으므로 실제 자동화에서는 `trap` 같은 복원 보장을 설계해야 한다. Test가 변경 전 code에서 실패한다면 현재 동작의 characterization이 아니라 기대 동작을 미리 적은 wishful test일 가능성이 있다.

## Stage 3: 변경

안전망을 만든 뒤 새로운 요구사항을 실패 test로 표현하고 구현과 refactoring을 진행한다.

Characterization test가 깨졌을 때는 기계가 test failure를 감지할 수 있지만 그 변화가 의도한 변경인지 회귀인지는 판단할 수 없다.

| 상황 | 판정과 처리 |
|---|---|
| 의도하지 않은 회귀 | 구현을 수정해 기존 characterization 유지 |
| 의도한 동작 변경 | 사람이 확인하고 characterization을 갱신하며 diff를 변경 기록으로 남김 |

전체 test 통과와 source 변경에 test 변경이 동반됐는지는 기계 gate로 확인할 수 있다. 의미 있는 test인지와 characterization 변경이 의도적인지는 사람이 판단한다.

## Stage 4: 적립

이번 작업에서 실제로 알게 된 module, 실행 명령, 위험 지대와 다음 사람이 알아야 할 사실만 `LANDING.md`에 추가한다. 손대지 않은 영역은 이해한 것처럼 문서화하지 않는다.

이 방식은 문서를 선불로 완성하려 하지 않는다. 여러 요청을 처리하면서 이동한 경로만 점진적으로 지도에 추가한다.

## 기계와 사람의 경계

| 기계가 맡기 좋은 판정 | 사람이 맡아야 하는 판정 |
|---|---|
| test command 성공 여부 | 깨진 characterization이 회귀인지 의도인지 |
| 변경 전 code에서 characterization 통과 여부 | test가 의미 있는 동작을 검증하는지 |
| source와 test file이 함께 변경됐는지 | 정당한 test 예외인지 |
| 변경 파일과 command evidence 기록 | 요구사항 해석과 최종 승인 |

Gate를 강하게 만드는 것보다 기계가 확실히 판정할 수 있는 범위를 넘지 않는 것이 중요하다.

## 개인 적용 체크리스트

- [ ] 전체 repository를 읽기 전에 요청의 entry point를 찾았는가
- [ ] 변경 지점과 공유 code의 회귀 후보를 기록했는가
- [ ] 기존 test baseline이 green인가
- [ ] 변경 전 code에서 characterization test가 통과하는가
- [ ] bug처럼 보여도 현재 동작을 먼저 그대로 고정했는가
- [ ] 깨진 characterization의 의도 여부를 사람이 확인했는가
- [ ] 실제로 알게 된 내용만 landing note에 추가했는가
- [ ] 자동 차단에는 loop guard와 탈출 경로가 있는가

## 한계

이 설계는 실사용 검증이 없는 제안이다.

> **Status: Outdated** (2026-08-11)
> Issue 0의 품질 baseline 단계는 한 차례 실행됐지만 Characterization Test와 기능 변경 단계는 아직 실사용 검증이 없다.

Local service 실행이 불가능하거나 외부 API, 시간, 난수, 비동기 timing에 크게 의존하면 characterization test가 어렵다. Test가 없는 repository의 최소 부트스트랩도 예상보다 큰 작업일 수 있다. Test-presence gate는 test file이 함께 바뀌었다는 사실만 확인할 뿐 RED를 실제로 관찰했거나 test가 의미 있다는 사실까지 증명하지 못한다.

## See Also

- [AI Agent 산문 게이트와 결정적 게이트](ai-agent-prose-vs-deterministic-gates.md)
- [AI Harness 실측과 선택 가이드](ai-harness-audit-and-selection.md)
- [AI Agent Teams와 Git Worktree](agent-teams-and-git-worktrees.md)
- [Superpowers Brownfield 실전 가이드](superpowers-brownfield-field-guide.md)
