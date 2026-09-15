# How to Write an Effective Software Design Document

> Source: https://refactoringenglish.com/excerpts/write-an-effective-design-doc/
> Collected: 2026-09-15
> Published: 2026-06-24

## Source record

Michael Lynch가 Google, Microsoft와 자신의 회사에서 설계 문서를 작성한 경험을 바탕으로 정리한 글이다. Refactoring English의 일부로 공개되었다. 저작권이 있는 원문 전문은 복제하지 않고, Wiki에서 근거를 확인하는 데 필요한 짧은 인용과 구조를 기록한다.

핵심 문제의식은 구현 전에 어려운 문제와 중요한 결정을 드러내고, 동료와 관련 팀이 의견을 줄 수 있게 하는 것이다. 원문은 “A good design doc can save you years of development time.”이라고 설명한다.

## When to write and how much

- 여러 사람이 구현을 조율하거나 여러 팀이 협업하는가?
- 장기간 개발·운영할 가능성이 있는가?
- 목표와 요구사항이 모호한가?
- 보안·법률처럼 설계 단계에서 막아야 할 큰 위험이 있는가?

복잡성과 위험이 클수록 설계 문서의 가치가 커진다. 문서 분량에는 보편적인 정답이 없으며 팀의 목표, 위험, 일정과 문화에 맞춰야 한다. 어떤 작업에는 설계 문서가 필요하지 않을 수도 있다.

세부 사항을 포함할지 판단하는 핵심 질문은 “what’s the penalty for being wrong?”이다. 나중에 바꾸기 어려운 언어·저장소·시스템 경계는 문서에서 다루고, 쉽게 되돌릴 수 있는 화면상의 작은 선택은 구현 단계로 남긴다.

## Components described by the source

- Context: Title, Metadata, Objective, Background, Related documents
- Scope and behavior: Goals, Non-goals, Scenarios
- Explanation: Diagrams, Glossary, Constraints
- Quality and operations: Service level objectives, Monitoring / alerting, Timeline, Interfaces, Dependencies / infrastructure, Security, Privacy, Legal considerations, Logging
- Decision history: Open issues, Resolved issues, Alternatives considered

모든 문서에 모든 section을 넣을 필요는 없다. 프로젝트에 필요한 subset을 고른다.

## Important guidance

- Objective는 누구나 이해할 수 있는 한 문장으로 첫 page에 둔다.
- Background는 문제, 동기와 과거 시도를 설명하며 외부 구두 설명 없이도 이해돼야 한다.
- Goals는 구현 수단보다 사용자·팀·회사의 변화로 작성한다.
- Non-goals는 독자가 scope로 오해할 항목을 명시적으로 제외한다.
- Scenarios는 완성된 system이 현실에서 어떻게 쓰이는지 보여준다.
- Diagram은 data flow, component 관계, dependency, protocol을 보여주고 수정 가능한 원본을 연결한다.
- SLO는 성능과 안정성 목표를 측정 가능한 값으로 바꾸며 monitoring은 production에서 이를 확인하는 방법을 정의한다.
- Security, privacy와 logging은 구현 이후가 아니라 설계 단계부터 다룬다.
- Open issue에는 문제, 선택지와 바로 다음 행동을 적는다.
- 결정한 issue는 논의를 버리지 않고 결론과 함께 Resolved issues로 옮긴다.
- Alternatives considered에는 유력했지만 채택하지 않은 대안과 짧은 이유를 남긴다.

## Review

초안을 작성한 뒤에는 팀에 공유해 의견을 받는다. 이 excerpt는 자세한 review 진행법을 별도 글로 연결하며 끝난다.
