# AOP가 필요한 상황 강의 원문

> Source: Inflearn 김영한, 스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술
> Collected: 2026-08-22
> Published: Unknown
> Transcript note: 사용자 제공 자동 전사에서 문장부호와 반복 발화를 정리한 보존본

자 이제 여러분 드디어 AOP까지 왔습니다.

AOP를 처음부터 이론적으로 공부하면 join point 같은 생소한 용어 때문에 어렵게 느껴질 수 있습니다. 하지만 AOP를 언제, 왜 쓰는지 예제로 먼저 이해하고 세부 이론과 용어를 나중에 배우면 어렵지 않습니다.

먼저 AOP가 필요한 상황을 보겠습니다. 모든 method의 호출 시간을 측정해야 한다고 가정합니다. Method가 아주 많다면 각각의 시작과 끝에 시간 측정 code를 모두 넣어야 합니다. 측정 단위를 바꾸라는 요구가 생기면 그 모든 code를 다시 수정해야 합니다.

예를 들어 `MemberService.join()`에 시간 측정 logic을 추가합니다. 시작 시각은 `System.currentTimeMillis()`로 얻고, business logic을 실행한 뒤 종료 시각에서 시작 시각을 뺍니다. Exception이 발생해도 측정해야 하므로 business logic을 `try`로 감싸고 종료 logic은 `finally`에 둡니다. 실행한 test에서는 회원 가입에 걸린 시간이 millisecond 단위로 출력됩니다.

하지만 회원 가입만 측정해서는 요구사항을 충족하지 못합니다. `findMembers()`를 포함한 모든 대상 method에 같은 시작·종료 code를 반복해서 넣어야 합니다. Method가 많다면 이 작업만으로도 매우 큰 반복이 됩니다.

회원 목록을 처음 호출했을 때는 시간이 더 오래 걸리고 다음 호출은 빨라질 수 있습니다. 첫 실행에는 class metadata loading 같은 초기 작업이 포함될 수 있기 때문입니다. 실제로 높은 성능이 필요한 server는 시작 후 여러 기능을 미리 호출하는 warm-up을 수행하기도 합니다.

여기서 더 중요한 문제는 회원 가입과 회원 조회 시간을 측정하는 기능이 핵심 business 기능이 아니라는 점입니다. `try` 안의 회원 가입과 조회는 핵심 business logic이고, 실행 시간을 측정하는 code는 여러 method에 공통으로 들어가는 부가 logic입니다.

이처럼 회원 업무 자체는 핵심 관심 사항(core concern), 여러 method에 공통으로 적용되는 시간 측정은 공통 관심 사항(cross-cutting concern)이라고 합니다.

시간 측정 logic과 핵심 business logic이 한 method 안에 섞이면 유지보수가 어렵습니다. 측정 code는 대상 method의 실행 전과 후를 함께 감싸야 하므로 전체 구조를 단순한 공통 method 하나로 추출하기도 쉽지 않습니다. 측정 방식을 변경하려면 반복해서 삽입한 모든 code를 찾아 수정해야 합니다.

즉 문제는 다음과 같습니다.

- 회원 가입과 회원 조회의 핵심 business logic에 시간 측정 code가 섞입니다.
- 같은 측정 code가 여러 method에 반복됩니다.
- 실행 전후를 감싸는 구조라 별도의 공통 method로 분리하기 어렵습니다.
- 측정 방식을 변경하면 적용한 모든 위치를 찾아 수정해야 합니다.

이 상황에서 공통 관심 사항을 핵심 관심 사항과 분리하기 위해 AOP를 사용할 수 있습니다. 구체적인 AOP 적용 방법은 다음 강의에서 설명합니다.

감사합니다.

