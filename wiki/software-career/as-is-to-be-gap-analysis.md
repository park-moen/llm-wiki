# AS-IS와 TO-BE GAP 분석

> Sources: International Institute of Business Analysis, Unknown; APQC, 2026-09; U.S. Department of Justice, Unknown
> Raw: [IIBA Strategy Analysis](../../raw/software-career/iiba-business-analysis-core-standard-current-future-state.md); [APQC Process Mapping](../../raw/software-career/apqc-current-and-future-state-process-mapping.md); [DOJ Architecture Definitions](../../raw/software-career/us-doj-as-is-to-be-gap-transition-plan.md)
> Updated: 2026-09-21

## Overview

AS-IS와 TO-BE GAP 분석은 현재 상태를 원하는 결과와 단순 비교하는 표가 아니다. 실제로 어떻게 일하는지 확인해 baseline을 만들고, business need가 충족된다고 판단할 수 있는 미래 조건을 정의한 뒤, 그 사이를 메우는 변화·우선순위·위험·전환 단계를 결정하는 전략 분석이다. IIBA는 이를 current state, future state, risk, change strategy를 연결하는 작업으로 설명한다. [IIBA Strategy Analysis](../../raw/software-career/iiba-business-analysis-core-standard-current-future-state.md)

## 네 가지 산출물

| 산출물 | 답해야 할 질문 | 포함할 내용 |
| --- | --- | --- |
| AS-IS | 지금 실제로 무엇이 일어나는가? | 흐름, 역할, system, rule, handoff, 예외, 의존성, 측정값과 문제 증거 |
| TO-BE | 어떤 조건이 갖춰지면 문제를 해결했다고 볼 수 있는가? | 목표, 성공 기준, 새로 만들고·바꾸고·없앨 capability와 예상 가치 |
| GAP | 무엇이 부족하거나 달라져야 하는가? | 현재와 목표의 차이, 원인, 영향 범위, 필요한 변화 |
| Transition plan | 어떤 순서로 GAP을 닫을 것인가? | action, owner, 우선순위, 선행 조건, 위험 완화, milestone |

AS-IS는 희망이나 문서상 절차가 아니라 **실제로 수행되는 일**을 기록해야 한다. APQC는 process step뿐 아니라 role, system, rule, handoff와 variation까지 포착하라고 권한다. TO-BE도 화면이나 solution을 먼저 그리는 문서가 아니라, business need를 만족하는 목표·조건과 바뀌어야 할 범위를 먼저 정하는 문서다. [APQC Process Mapping](../../raw/software-career/apqc-current-and-future-state-process-mapping.md); [IIBA Strategy Analysis](../../raw/software-career/iiba-business-analysis-core-standard-current-future-state.md)

## 분석 순서

1. **문제와 범위를 확정합니다.** 목표, 시작 event, 종료 결과, 이해관계자, 제약, upstream·downstream 연결을 정합니다. 범위가 없으면 GAP이 모든 불만의 목록이 됩니다.
2. **AS-IS를 evidence로 기록합니다.** 실제 workflow를 end-to-end로 그리고, handoff·예외·중복·대기·불명확한 책임과 현재 성과를 확인합니다.
3. **TO-BE의 성공 조건을 정합니다.** business goal, stakeholder requirement, performance gap을 근거로 목표 상태를 정의합니다. 이때 새 기능뿐 아니라 없앨 단계, 바뀔 rule·책임·control도 적습니다.
4. **같은 관점으로 차이를 비교합니다.** process, role, data, system, rule, measure, dependency별로 AS-IS와 TO-BE를 나란히 놓아 GAP을 명시합니다.
5. **전환 전략으로 바꿉니다.** GAP마다 대안·가치·위험·실행 난이도를 검토하고 action과 순서를 결정합니다. 전환이 길거나 불확실하면 intermediate state를 별도로 둡니다.
6. **당사자와 검증합니다.** process owner·실무자·subject-matter expert·인접 process 담당자가 실제와 목표를 검토하고, 측정값으로 변화 후 결과를 확인합니다.

## 실무용 GAP 표

아래 표는 기능 목록 대신 변화의 근거와 완료 판단을 함께 남기기 위한 최소 형식이다.

| 관점 | AS-IS evidence | TO-BE 조건 | GAP | 전환 action | Owner | 성공 측정 |
| --- | --- | --- | --- | --- | --- | --- |
| Process | 현재 흐름과 예외 | 목표 흐름과 종료 조건 | 제거·추가·변경할 단계 | process 변경 작업 | 책임자 | cycle time 또는 오류 흐름 |
| Role | 현재 승인·수작업 책임 | 명확한 책임과 handoff | ownership 공백 또는 중복 | 역할·권한 조정 | process owner | handoff 지연 또는 재작업 |
| System/Data | 현재 system, data source, interface | 필요한 capability와 data 품질 | integration, migration, access 차이 | 설계·구현·migration | 기술 책임자 | 실패율 또는 data 품질 |
| Rule/Control | 현재 policy, validation, audit | 목표 rule과 control | 적용되지 않거나 과도한 rule | policy·validation 변경 | 업무·보안 책임자 | 위반 또는 예외 처리 |

표의 `GAP` 칸은 “새 기능이 필요함”처럼 solution을 먼저 쓰지 않는다. 예를 들어 “담당자가 handoff 뒤 상태를 알 수 없음”처럼 관찰된 현재 문제와 목표 조건의 차이를 적고, `전환 action`에서 notification·workflow 변경·role 재배치 같은 대안을 비교한다.

## 흔한 실패와 방지 기준

- **AS-IS를 이상적인 절차로 작성함**: 실제 담당자와 함께 예외·우회·대기 시간을 확인합니다.
- **TO-BE를 tool 도입으로 축소함**: 목표 outcome과 measure를 먼저 합의하고, tool은 그 조건을 충족하는 대안으로 다룹니다.
- **GAP을 feature backlog로만 만듦**: process·role·data·rule·dependency도 같은 표에서 비교합니다.
- **전환 비용을 생략함**: transition action마다 owner, 선행 조건, risk와 milestone을 둡니다. 미국 법무부의 정의도 transition plan을 action·priority·milestone을 포함해 AS-IS와 TO-BE를 잇는 blueprint로 본다. [DOJ Architecture Definitions](../../raw/software-career/us-doj-as-is-to-be-gap-transition-plan.md)
- **검증 지표가 없음**: 처음에 정한 business outcome을 보여 줄 소수의 measure를 유지합니다. APQC는 많은 dashboard보다 집중된 지표 집합이 더 유용하다고 설명한다. [APQC Process Mapping](../../raw/software-career/apqc-current-and-future-state-process-mapping.md)

## 개발 문서에 적용하기

제품이나 시스템 변경에서는 AS-IS를 code와 운영 evidence로, TO-BE를 요구사항과 acceptance criteria로, GAP을 구현·migration·운영 변경의 묶음으로 연결한다. 이 연결이 있어야 설계 문서가 현재 구현을 설명하고, task가 왜 필요한지와 무엇으로 완료를 판정하는지를 함께 보여 줄 수 있다.

## See Also

- [효과적인 Software Design Document 작성법](effective-software-design-document.md)
- [Jira 작업 항목의 분할 크기와 관리 비용](work-item-decomposition-and-tracking-granularity.md)
