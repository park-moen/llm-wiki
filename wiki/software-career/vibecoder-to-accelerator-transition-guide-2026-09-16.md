# Vibecoder에서 Accelerator로 전환하는 실천 가이드

> Sources: [AI Coding에서 Code Reading과 Intent 보존](ai-coding-code-reading-and-intent-preservation.md); [Claude Code 팀의 AI-Native 개발 Workflow와 개발자 역량](claude-code-team-ai-native-development-workflow.md); [취업 초기 주니어를 위한 AI-Native 개발 프로세스](junior-ai-native-development-process.md); [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](ai-coding-software-fundamentals.md)
> Archived: 2026-09-16

## Overview

Vibecoding에서 accelerator 방식으로 전환할 때 AI 사용량 자체를 줄일 필요는 없다. 먼저 줄여야 하는 것은 이해하지 못한 변경의 크기와 생성 속도다. AI가 code를 작성하더라도 사람이 요구사항·설계·diff·검증을 이해하고 이후 변경과 장애 복구를 책임질 수 있다면 accelerator에 가깝다. 전환의 핵심은 manual coding으로 돌아가는 것이 아니라 AI의 생성 속도를 자신의 review와 integration capacity 안에 두는 것이다.

## 병렬 작업보다 이해 가능한 한 조각을 우선한다

익숙하지 않은 영역에서는 다음을 기본값으로 둔다.

- 한 번에 feature 하나만 다룬다.
- Write agent는 하나만 사용한다.
- 현재 branch에서 작은 변경부터 진행한다.
- Subagent는 조사와 review 같은 읽기 전용 역할부터 사용한다.
- Worktree 병렬 구현은 codebase와 domain이 익숙한 영역으로 제한한다.

사람의 review 속도보다 AI의 생성 속도가 빨라지면 `cognitive debt`가 쌓인다. 높은 처리량을 유지하는 것보다 모든 변경을 설명하고 통합할 수 있는 크기로 낮추는 것이 먼저다.

## 구현 전에 자신의 이해를 먼저 적는다

완전한 설계 문서를 만들 필요는 없다. 구현 요청 전에 다음 항목을 짧게 작성한다.

```text
해결하려는 문제:
정상 동작:
실패 동작:
관련될 것으로 예상하는 file과 흐름:
완료를 증명할 test:
아직 모르는 점:
```

이 기록은 AI가 만든 plan을 평가할 기준이 된다. 자신의 예상이 하나도 없는 상태에서 AI의 설명을 받으면 그럴듯한 답과 repository의 실제 구조를 구분하기 어렵다.

## AI에게 구현보다 조사를 먼저 맡긴다

첫 요청에서는 file을 수정하지 못하게 하고 codebase의 근거를 찾게 한다.

```text
아직 수정하지 마세요.

이 요구사항과 관련된 entry point, 호출 흐름, domain rule,
수정 후보 file과 기존 test를 조사하세요.

제 예상과 다른 부분, 아직 결정하지 않은 사항,
잘못 이해했을 가능성이 있는 가정을 구분해서 설명하세요.
각 설명에는 근거 file을 붙이세요.
```

AI가 제시한 file을 IDE에서 직접 열고 같은 호출 경로를 따라간다. 설명과 실제 code가 다르면 code를 정본으로 삼는다.

## Plan을 이해한 뒤 작은 Behavior만 구현한다

다음 질문에 답할 수 있을 때 구현으로 넘어간다.

- 어떤 사용자 또는 system behavior가 달라지는가?
- 어느 layer와 public interface가 영향을 받는가?
- AI가 새롭게 정한 domain rule은 없는가?
- 어떤 test가 성공을 증명하는가?
- 회귀 가능성이 있는 영역은 어디인가?

구현도 feature 전체가 아니라 작은 behavior 하나씩 맡긴다.

```text
확정한 plan 중 첫 번째 behavior만 구현하세요.
관련 없는 refactoring은 하지 마세요.
실패하는 test를 먼저 만들고, 통과할 만큼만 수정하세요.
완료 후 변경 file과 각 변경의 이유를 설명하세요.
```

Accelerator의 생산성은 모든 code를 직접 입력하는 데서 나오지 않는다. 사람이 문제·scope·가정·interface와 검증 방식을 먼저 정렬하고, AI가 그 이해를 빠르게 code로 옮기게 하는 데서 나온다.

## Diff와 설계 판단을 직접 소유한다

AI의 완료 요약만 읽지 않고 모든 변경 file을 직접 연다. 각 변경에서 다음을 확인한다.

- 이 변경이 없으면 어떤 test가 실패하는가?
- 이 책임이 왜 해당 class나 function에 있는가?
- 기존 convention과 일치하는가?
- Input·output·state·failure mode는 무엇인가?
- AI가 근거 없이 선택한 값이나 구조는 없는가?
- 기존 test를 삭제하거나 약화하지 않았는가?

모르는 부분은 AI에게 설명시키되 근거 file과 기존 구현을 함께 요구한다.

```text
이 설계가 기존 architecture와 일치한다는 근거를 보여주세요.
관련 interface와 기존 구현을 비교해서 설명하세요.
추측한 부분은 추측이라고 표시하세요.
```

AI의 설명은 검토 후보이며 정답이 아니다. 설명이 실제 repository와 일치하는지 사람이 확인한다.

## 매 작업에서 일부는 직접 다룬다

모든 code를 따라 입력할 필요는 없다. 대신 매 작업에서 다음 중 하나는 직접 수행한다.

- Test case 추가
- 작은 refactoring
- 이름이나 interface 개선
- 실패 원인 추적
- AI가 세운 잘못된 가정 수정
- 요구사항 변경에 따른 후속 수정
- Debugger를 이용한 실제 data flow 확인

Test가 실패했을 때는 로그를 바로 AI에게 넘기기 전에 자신의 가설을 먼저 만든다.

```text
관찰한 증상:
가능한 원인:
각 원인을 구분할 실험:
가장 먼저 확인할 file:
```

그다음 AI에게 가설을 비판하거나 빠진 가능성을 찾게 한다. AI가 root cause를 대신 생각하게 하는 것과 자신의 가설을 검증하는 데 사용하는 것은 다른 학습 방식이다.

## 완료 기준에 설명 가능성을 포함한다

작업 종료 전에 다음을 확인한다.

- 요구사항의 정상·실패 동작을 설명할 수 있다.
- Entry point부터 database 또는 외부 API까지 호출 흐름을 설명할 수 있다.
- AI가 변경한 모든 file을 읽었다.
- Test가 어떤 behavior를 검증하는지 설명할 수 있다.
- 중요한 설계 결정과 대안을 설명할 수 있다.
- 장애가 발생하면 어디서부터 조사할지 안다.
- 작은 요구사항 변경 하나는 AI 없이 직접 처리할 수 있다.

Test가 통과해도 이 항목을 설명하지 못한다면 구현은 완료됐지만 학습과 인수인계는 끝나지 않은 상태다.

## AI에 맡길 일과 사람이 소유할 일을 구분한다

AI에는 다음 작업을 적극적으로 맡길 수 있다.

- Codebase 검색과 관련 file 후보 탐색
- 반복적인 boilerplate 작성
- Test 초안과 누락된 case 제안
- API 사용법과 문법 설명
- Diff의 위험 후보 검토
- 가설에 대한 반론 제시
- 문서와 code의 불일치 탐색
- Formatting·lint·build 같은 반복 가능한 작업

사람은 다음 판단을 계속 소유한다.

- 요구사항과 domain rule
- Public interface와 data model
- Architecture trade-off
- Security·migration·운영 위험
- Test가 실제 요구사항을 증명하는지에 대한 판단
- Production 반영과 장애 대응 책임

AI에게 구현 방법을 물어보는 것은 문제가 아니다. 그 구현이 왜 맞는지를 판단하는 일까지 넘기면 accelerator의 경계를 벗어난다.

## 지속적인 전환 기준

목표를 AI 없이 많은 code를 작성하는 것으로 잡지 않는다. AI와 함께 작업하면서 다음 경험을 반복해서 쌓는다.

- AI가 만든 code를 자신의 말로 설명한다.
- AI의 잘못된 가정을 발견한다.
- 직접 root cause를 추적한다.
- Test가 실제 defect를 잡는 경험을 만든다.
- 작은 refactoring이나 구현을 직접 수행한다.
- 새로 알게 된 domain 지식과 설계 근거를 기록한다.

장기적인 성장 기준은 AI 사용 전보다 더 많은 code를 생산하는지가 아니다. 더 복잡한 변경을 이해하고, AI가 실패했을 때 독립적으로 검증하고 복구하며, production 결과를 책임질 수 있게 되는지가 핵심이다.

## See Also

- [AI를 활용한 개발자 성장과 Career 판단](ai-assisted-engineering-growth-and-career-judgment.md)
- [AI 시대의 Software Engineering 학습 전략](ai-era-software-engineering-learning-strategy-2026-08-23.md)
- [AI Coding Autonomy Experiment와 Human-in-the-loop](../ai-agents/ai-coding-autonomy-experiment.md)
