# Claude Code ELI5 Skill로 쉬운 시각 설명 만들기

> Sources: 갓대희의 작은공간, 2026-08-24
> Raw: [Claude Code ELI5 Skill 튜토리얼](../../raw/ai-agents/2026-08-24-claude-code-eli5-skill-tutorial.md)
> Updated: 2026-09-15

## Overview

Claude Code ELI5 Skill은 새로운 추론 능력이나 실행 도구를 추가하지 않는다. 배경지식이 없는 사람을 청자로 정하고, 큰 그림과 짧은 설명으로 구성된 HTML 결과물을 만들도록 출력 형식을 고정한다. 모듈 구조, 설계 선택과 장애 흐름을 처음 파악할 때 유용하지만, 보기 쉬운 그림이 사실 검증을 대신하지는 않는다.

## 핵심은 능력 추가가 아니라 설명 계약이다

ELI5의 본체는 짧은 `SKILL.md`다. 별도의 MCP server나 실행 script 없이 다음 두 조건을 반복해서 적용한다.

| 조건 | 고정하는 내용 | 기대 효과 |
|---|---|---|
| 청자 | 주제를 전혀 모르는 사람 | 전문 용어와 사전 지식 가정을 줄인다. |
| 결과 형식 | 큰 그림과 적은 글로 만든 HTML | 긴 문단보다 전체 흐름을 먼저 보여준다. |

이름은 “5살에게 설명하기”라는 익숙한 표현을 쓰지만, 실제 본문은 어린아이 같은 말투보다 배경지식이 없는 사람을 위한 설명을 요구한다. 따라서 이 Skill을 단순한 말투 변환기가 아니라 **청자와 표현 매체를 고정한 설명 template**으로 보는 편이 정확하다.

이 사례가 보여주는 일반 원칙도 단순하다. Agent의 성능을 높이는 방법이 반드시 새 tool이나 긴 prompt일 필요는 없다. 자주 반복하는 작업에서 청자, 산출물과 품질 기준을 작게 고정해도 결과의 일관성을 높일 수 있다.

## 언제 쓰면 좋은가

ELI5는 정답을 확정하기 전, 전체 구조를 빠르게 이해하는 첫 단계에 적합하다.

- 처음 보는 module이 요청을 어떻게 처리하는지 파악할 때
- 특정 설계를 선택한 이유와 trade-off를 공유할 때
- 장애가 어느 component를 거쳐 전파됐는지 설명할 때
- 신규 입사자 onboarding이나 code review 전에 공통 그림을 만들 때

요청 범위는 작고 구체적이어야 한다. “프로젝트 전체를 설명해 줘”보다 file, module, 결정 또는 장애 하나를 지정해야 일반론으로 흐를 가능성이 줄어든다.

```text
/eli5 src/auth/middleware.ts가 요청마다 검사하는 흐름을 설명해 줘
```

```text
/eli5 지난 장애에서 timeout이 어떤 component를 거쳐 전파됐는지 설명해 줘. 관련 log도 읽어 줘
```

## 설치와 사용

원문 작성 시점에는 first-party 공식 plugin이 아니라 Anthropic의 community marketplace에서 배포되는 plugin이었다. “Anthropic 직원이 만들었다”와 “Anthropic 공식 plugin이다”를 같은 뜻으로 해석하면 안 된다.

Claude Code에서 community marketplace를 추가하고 plugin을 설치한다.

```text
/plugin marketplace add anthropics/claude-plugins-community
```

```text
/plugin install eli5@claude-community
```

설치 후 `plugin list`에서 상태를 확인한다. 정식 호출명은 `/eli5:eli5`이며, 이름 충돌이 없으면 짧은 `/eli5`도 사용할 수 있다.

Marketplace를 추가할 수 없는 환경에서는 같은 지침을 project의 `.claude/skills/eli5/SKILL.md`에 둘 수 있다. 한 번만 시험한다면 핵심 지침을 일반 prompt에 붙여 넣어도 된다. 차이는 모델 능력이 아니라 재사용, 팀 공유와 update 방식에 있다.

## 결과를 읽는 법

처리 흐름은 다음과 같다.

```text
topic 입력
  → Skill 본문과 topic을 context에 추가
  → 현재 대화와 project file을 참고
  → HTML explainer 생성
  → 지원 환경에서는 Artifact 게시, 아니면 local HTML 보존
```

HTML이 만들어진 것과 Artifact URL이 발급된 것은 별개의 성공 조건이다. 게시 기능은 account, plan, login 방식과 실행 환경에 영향을 받을 수 있다. URL이 없더라도 local HTML이 생성됐다면 설명 단계는 성공했을 수 있다.

또한 시각적 완성도와 사실 정확성도 분리해서 평가해야 한다. 도형과 흐름이 자연스러워 보여도 누락된 예외, 잘못 읽은 code나 추측한 인과관계가 있을 수 있다.

## 검증이 필요한 경계

ELI5는 이해 보조 수단이지 검증 도구가 아니다. 공개된 Skill에는 인용 확인, test 실행, code 수정과 security audit 절차가 포함되지 않는다.

다음 작업은 ELI5 결과만으로 끝내면 안 된다.

- 정확한 spec, 수치와 quota 확인
- edge case와 예외 분기 검증
- security, 법률·의료·금융 판단
- code 수정과 PR 생성

실무에서는 ELI5로 전체 지도를 만든 뒤 code, log, test와 공식 문서로 세부 사실을 확인한다. 설명 자료에 출처와 미확인 항목을 표시하면 “이해하기 쉬움”을 “검증됨”으로 오해하는 일을 줄일 수 있다.

## 보안과 배포 판단

Community marketplace는 first-party 공식 배포 경로와 신뢰 수준이 다르므로 설치 전에 source와 manifest를 확인한다. 이 ELI5 Skill 자체에는 실행 script가 없지만, 읽어 들인 사내 code, 고객명, key와 내부 URL이 HTML에 나타날 수 있다.

따라서 공유하기 전에 다음을 확인한다.

1. HTML 원문에 secret과 내부 정보가 없는가?
2. Artifact가 private인지, 조직 밖 공유가 가능한 설정인지 확인했는가?
3. 설명 속 핵심 흐름을 code와 log로 대조했는가?
4. 공유 대상의 눈높이에 맞지만 필요한 technical detail까지 지우지는 않았는가?

## See Also

- [Claude Code Hooks](claude-code-hooks.md)
- [im-not-ai Humanize Korean Skill 사용 가이드](im-not-ai-humanize-korean-skill-guide.md)
- [Agent Harness의 구조와 Deterministic Control Loop](agent-harness-anatomy-and-deterministic-control-loop.md)
