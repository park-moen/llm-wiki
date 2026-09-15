# AI를 활용한 개발자 성장과 Career 판단

> Sources: Pasha interview·Beyond Coding (Unknown)
> Raw: [From Backend Engineer to Head of Mobile transcript](../../raw/software-career/from-backend-engineer-to-head-of-mobile-lessons-uber.md)
> Updated: 2026-08-16

## Overview

Backend engineer에서 mobile engineer와 Head of Mobile로 이동한 Pasha의 인터뷰는 AI 활용, fundamentals, 협업과 career 선택을 하나의 성장 문제로 연결한다. 핵심은 AI를 자율 대행자로 내버려 두는 것이 아니라 spec·code·review를 함께 다루는 동료로 사용하면서, 사람이 fundamentals와 최종 판단을 소유하는 것이다. Career 성장은 문법 암기에서 끝나지 않고 제품의 이유, 팀의 의사결정, API 경계, migration과 지속 가능한 작업 속도까지 확장된다.

## AI를 Junior·Medium 동료처럼 사용한다

Pasha는 AI를 모든 일을 넘기는 대상보다 idea를 주고받고 spec과 code를 review하며 일부 구현을 돕는 junior·medium 동료에 가깝게 사용한다. 사람은 AI가 작성한 spec을 반복해 다듬고 code를 읽으며, review comment도 즉시 받아들이지 않고 실제 문제인지 판단한다.

```text
사람: 문제·spec·architecture·최종 판단
AI: 탐색·초안·구현·review 후보
Code·test: AI와 사람의 공통 검증 대상
```

이 workflow에서 code를 직접 입력하는 양은 줄어들 수 있지만 code reading과 review의 비중은 커진다. AI output을 읽고 설명할 수 없다면 위임의 범위가 학습 범위를 앞지른 상태다.

## Context 문서는 정답 파일이 아니다

프로젝트의 주 지침, architecture, analytics와 issue reporting 규칙을 별도 Markdown으로 제공하면 AI가 관련 작업에서 필요한 context를 찾기 쉬워워진다. 초기 작성 비용은 있지만 팀의 반복 설명을 줄이고 일관된 output을 유도할 수 있다.

다만 AI에게 이 문서 생성을 맡기면 codebase에 없는 architecture pattern을 적을 수 있다. 사람은 문서와 실제 code가 일치하는지 review하고, feature 변경으로 규칙이 달라졌다면 문서도 함께 갱신한다. Context file은 codebase의 대체물이 아니라 codebase를 설명하는 검증 가능한 지도여야 한다.

## 역할별 Parallel Context는 하나의 Feature에도 쓸 수 있다

인터뷰의 parallel workflow는 여러 feature를 동시에 작성하는 방식만을 뜻하지 않는다. 하나의 feature에서도 planner, implementer와 reviewer를 서로 다른 context로 분리할 수 있다.

```text
Planner context
└── spec과 작업 경계

Implementer context
└── 확정된 spec에 따른 구현

Reviewer context
└── spec과 diff의 불일치 검토
```

역할 context를 분리해도 사람은 각 pane의 책임과 현재 상태를 기억하고 결과를 통합해야 한다. 다른 feature를 동시에 수정할 때는 별도 repository clone이나 Worktree처럼 file·branch 상태를 격리하는 수단이 추가로 필요하다. 역할 분리와 Git 격리는 서로 다른 문제다.

## Fundamentals가 있어야 AI로 다른 Stack을 배울 수 있다

Pasha는 한 mobile ecosystem의 고수준 구조와 feature 통합 방식을 알고 있었기에, 덜 익숙한 Android SDK와 pattern의 초기 구현을 AI에게 도움받을 수 있었다. 여기서 전이되는 것은 framework 문법이 아니라 책임 분리, data flow, UI와 platform 제약을 보는 기초 사고다.

초보자가 바로 전체 application을 vibe coding하면 작동하는 결과는 빨리 얻을 수 있지만, 왜 그렇게 설계됐는지를 배울 기회를 놓칠 수 있다. 따라서 fundamentals를 먼저 읽고, AI가 만든 code의 방식을 끝까지 파고드는 순서가 필요하다.

## Career 성장은 기술 스택 밖으로 확장된다

### 모르면 모른다고 말한다

주니어가 이해하지 못한 내용을 이해했다고 넘어가면 오해가 code와 의사결정에 남는다. Domain expert나 선임에게 설명을 다시 요청하는 것은 능력 부족의 신호가 아니라 팀의 공통 mental model을 맞추는 행동이다.

### 모든 의견을 싸움으로 만들지 않는다

의견을 낼 때는 제품, 사용자, security, 유지보수성에 실제 차이를 만드는지 본다. 모든 사소한 차이에 맞서면 정말 중요한 반대의 신뢰도가 낮아진다. 의견을 충분히 제시한 뒤 팀이 다른 결정을 내리면 그 결정을 실행하고, 새 evidence가 생겼을 때 다시 논의한다.

### 사람은 약점이 없어서가 아니라 강점으로 기여한다

팀은 같은 형태의 개발자를 복제하는 곳이 아니다. 구현, debugging, product 감각, communication, domain 지식과 운영 경험처럼 서로 다른 강점을 결합해 개인의 약점을 팀으로 보완한다.

### 만들 수 있는가와 만들 가치가 있는가를 분리한다

구현 난이도가 높아도 사용자가 필요로 하지 않는 product라면 노력이 영향으로 이어지지 않는다. Career를 선택할 때도 stack 이름만 보지 말고 그 회사에서 자신의 기술이 사용자 가치의 핵심인지, 실제 feedback을 받을 수 있는지를 본다.

## 장기 System에서는 변경 비용이 핵심이다

인터뷰의 대규모 rewrite와 payment SDK migration 경험은 새 code를 만드는 것보다 기존 사용자와 team을 이동시키는 일이 더 오래 걸릴 수 있음을 보여준다. 새 interface가 더 낫더라도 직접적인 매출 증가로 설명하기 어렵고, 기존 호출자는 예전 방식에 익숙하며, 두 경로를 함께 유지하는 비용이 생긴다.

이 경험은 public API를 처음부터 필요 이상으로 넓게 열지 않는 이유와도 연결된다. 외부에 노출된 선택지는 나중에 닫기 어렵다. 복잡성을 단순한 interface 뒤에 숨기고 강한 사용 사례가 확인될 때만 외부 권한을 늘리는 방식이 장기 변경 비용을 줄인다.

## 지속 가능한 속도도 Engineering 판단이다

큰 rewrite나 deadline이 있더라도 밤샘과 burnout을 기본 전제로 삼지 않는다. 팀과 제품이 장기적으로 내야 할 비용을 개인의 소진으로 숨기면 일정은 맞출 수 있어도 학습, 품질과 인력을 잃을 수 있다. 중요한 것은 일정을 무시하는 것이 아니라 일정·범위·품질·사람의 건강 사이의 trade-off를 숨기지 않는 것이다.

## AI 시대의 Junior 적용 순서

1. AI에게 새 codebase의 feature entry point와 관련 file을 찾게 한다.
2. 제시된 file을 IDE에서 열어 실제 호출 경로를 따라간다.
3. Planner와 reviewer context를 구현 context와 분리하되 한번에 하나의 feature를 중심으로 운영한다.
4. AI가 작성한 spec·context 문서·code를 실제 repository와 대조한다.
5. 새 framework의 문법보다 input·output·state·failure mode를 먼저 이해한다.
6. 이해하지 못한 domain과 설계 결정은 선임에게 다시 질문한다.
7. 변경이 사용자 가치, 유지보수 비용과 팀의 장기 속도에 미치는 영향을 설명한다.

## 한계

원문은 영어 자동 자막 transcript이므로 고유명사, 수치와 문장 경계에 오인식이 있을 수 있다. AI 생산성, engineering 채용 시장, mobile career와 cross-platform 기술에 대한 발언은 인터뷰이의 개인 경험과 의견이며 보편적 예측이 아니다. 특히 출처나 수치가 불분명하게 언급된 조사는 독립적 근거로 사용하지 않는다.

## See Also

- [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md)
- [Claude Code 팀의 AI-Native 개발 Workflow와 개발자 역량](claude-code-team-ai-native-development-workflow.md)
- [Software Engineering을 Time·Scale·Trade-off로 이해하기](software-engineering-time-scale-tradeoffs.md)
- [AI Agent Teams와 Git Worktree](../ai-agents/agent-teams-and-git-worktrees.md)
