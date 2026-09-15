# Claude Code ELI5 스킬 이란? : Anthropic에서 쓰던 화제의 ELI5 스킬(ELI5 설치 및 사용해보기)

> Source: https://goddaehee.tistory.com/633
> Collected: 2026-09-15
> Published: 2026-08-24

## 원문 보존 범위

원문은 갓대희의 작은공간에 게시된 Claude Code ELI5 Skill 설치·사용 튜토리얼이다. 저작권이 있는 페이지 전문은 복제하지 않고, Wiki의 근거 확인에 필요한 원문 문구, 명령, metadata와 글의 논지를 보존한다. 전체 문맥과 실행 화면은 Source URL에서 확인한다.

## 원문의 기준과 대상

- 글은 2026년 8월 24일을 기준으로 작성되었다.
- 원문 게시자는 Claude Code 팀의 Thariq Shihipar가 2026년 8월 21일(UTC)에 ELI5 Skill을 공개했다고 설명한다.
- 글에서 다루는 대상은 `eli5@claude-community`이며, 작성자는 Thariq Shihipar다.
- 2026년 8월 24일 기준 공식 plugin이 아니라 `anthropics/claude-plugins-community`에서 배포되는 community plugin으로 설명한다.
- 원문은 “Anthropic 직원이 만들었다”와 “Anthropic 공식 plugin이다”를 같은 뜻으로 해석하면 안 된다고 구분한다.
- `plugin.json`의 source metadata는 name `eli5`, version `1.0.0`, author `Thariq Shihipar`, license `MIT`로 기록되어 있다.

## 공개된 Skill 본문

```markdown
---
name: eli5
description: Explain a topic like I'm a 5 year old. Use when the user types /eli5 <topic> or asks for a dead-simple picture explainer of how something works.
---

# eli5

Explain like I'm someone who knows nothing about this topic, using a HTML artifact with big pictures and few words.

Topic: $ARGUMENTS
```

원문은 이 파일을 10줄, 321바이트라고 기록한다. 별도의 MCP server나 실행 script는 없으며, 설명 대상의 눈높이와 출력 형식을 고정하는 지침이라고 해석한다.

## 설치와 호출

Claude Code session의 slash command:

```text
/plugin marketplace add anthropics/claude-plugins-community
```

```text
/plugin install eli5@claude-community
```

```text
/plugin list
```

정식 호출명은 `/eli5:eli5`이고, 이름 충돌이 없으면 `/eli5`도 쓸 수 있다고 설명한다.

```text
/eli5 how does DNS work
```

작성자가 제시한 활용 예:

```text
/eli5 how does this module work
/eli5 why did we make this tradeoff
/eli5 what caused this incident
```

Marketplace 설치가 어렵다면 `.claude/skills/eli5/SKILL.md` 또는 `~/.claude/skills/eli5/SKILL.md`에 Skill을 직접 둘 수 있다. 이 경우 community marketplace의 자동 update는 받지 못한다.

## 동작 방식에 대한 원문 설명

1. 사용자가 `/eli5 <topic>` 또는 `/eli5:eli5 <topic>`을 호출한다.
2. Skill 본문이 context에 들어가고 `$ARGUMENTS`에 topic이 전달된다.
3. Claude가 현재 대화와 project file을 참고해 HTML explainer를 만든다.
4. 환경이 지원하면 Artifact로 발행하고, 지원하지 않으면 local HTML file로 남을 수 있다.

원문은 HTML Artifact 게시 여부가 plan, login 방식과 Claude Code CLI version에 따라 달라진다고 설명한다. 게시가 되지 않아도 HTML 생성까지 성공했다면 Skill 전체의 실패와 구분해야 한다고 본다.

## 원문이 제시한 활용과 한계

잘 맞는 용도:

- 처음 보는 module의 전체 흐름 파악
- 설계 trade-off 설명
- 장애 원인과 전파 경로 설명
- 신규 입사자 onboarding, code review, postmortem의 첫 설명 자료

맞지 않는 용도:

- 정확한 spec, 숫자와 quota 확인
- 예외 분기와 edge case 검증
- security review, 법률·의료·금융 판단
- code 수정이나 PR 생성

원문은 시각적으로 그럴듯한 결과가 사실의 정확성을 보장하지 않는다고 강조한다. 전체 구조를 빠르게 이해하는 첫 화면으로 사용하고, 세부 사실은 code와 공식 문서에서 다시 확인하라고 권한다.

## 보안과 운영상 주의

- Community marketplace를 추가하기 전에 source를 확인해야 한다.
- ELI5 자체에는 script가 없지만 설명 대상 code의 secret이 HTML에 포함될 수 있다.
- Artifact URL의 공개·공유 범위는 plan과 조직 설정에 따라 달라진다.
- Plugin 자체의 별도 가격표는 없으며 Claude plan과 token 사용량이 적용된다.
- 프로젝트 전체보다 module 하나나 장애 하나처럼 범위를 좁혀 요청하는 편이 낫다.

## 원문 목차

1. 왜 지금 이 스킬인가
2. 공식 플러그인인가? 동명 스킬과 구별
3. 이렇게 짧은 스킬이 왜 잘 먹히나
4. 준비물
5. 실제로 따라해보기
6. 기존 프로젝트에서 활용하는 법과 대체 경로
7. 한계, 오류, 보안, 대안
8. Q&A와 정리
9. 참고 자료
