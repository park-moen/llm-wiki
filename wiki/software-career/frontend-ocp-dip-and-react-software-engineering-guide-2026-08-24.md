# Frontend OCP·DIP와 React 설계 패러다임 통합 가이드

> Sources: [Frontend에서 OCP와 의존성 역전을 적용하는 실무 가이드](frontend-ocp-and-dependency-inversion-practical-guide-2026-08-24.md); [Spring Bean과 의존관계 설정](../spring/spring-beans-and-dependency-injection.md); [AI 시대의 Software Engineering 학습 전략](ai-era-software-engineering-learning-strategy-2026-08-23.md); [Superpowers 기반 Brownfield 연습 워크플로 초기 설계](../ai-agents/superpowers-brownfield-practice-workflow-initial-design.md); [AI 중심 실무 환경의 Full-stack 개발자 6개월 학습 로드맵](ai-native-fullstack-learning-roadmap-2026-08-23.md)
> Archived: 2026-08-24

## Overview

소프트웨어 공학은 OOP 하나로 구성된 체계가 아니라 객체지향·함수형·절차형·선언형 방식을 문제에 맞게 조합해 복잡성을 관리하는 분야다. React는 소프트웨어 공학의 흐름을 거부한 것이 아니라, 상태에서 UI를 계산하고 화면을 트리로 조립하는 문제에 함수와 합성이 더 적합하다고 선택했다. 따라서 Java·Spring의 OCP와 DIP는 Frontend에서도 유효하지만 `interface + Impl class + DI container`라는 모양보다 TypeScript type·함수, adapter, props·Context와 component composition으로 나타난다.

## 먼저 바로잡을 전제: 소프트웨어 공학은 OOP만의 체계가 아니다

Enterprise application과 Java 생태계에서 OOP가 큰 비중을 차지해 온 것은 사실이지만, 소프트웨어 공학 전체가 OOP 위에만 세워진 것은 아니다. 운영체제와 system programming에는 절차형 접근이 강하고, 관계형 database와 SQL은 선언형이며, data transformation과 동시성에는 함수형 사고가 자주 쓰인다. 현대 application은 이 패러다임들을 한 시스템 안에서 함께 사용한다.

따라서 React를 이해할 때 질문을 다음처럼 바꾸는 편이 좋다.

```text
왜 React는 OOP를 따르지 않는가?
→ React가 해결하는 문제에는 어떤 패러다임이 더 적합한가?
```

React가 class hierarchy를 중심에 놓지 않는다는 사실은 설계 원칙이 없다는 뜻이 아니다. 책임, 상태 소유권, 의존성, 캡슐화, public API와 변경 전파를 다른 도구로 다룬다.

## React의 핵심 문제는 객체 협력보다 상태에서 UI를 계산하는 것이다

Spring의 핵심 모델은 여러 객체의 협력에 가깝다.

```text
Controller → Service → Repository
```

각 객체가 역할과 행위를 가지고 다른 객체에 메시지를 보내며, DI container가 객체의 생성과 연결을 관리한다.

React의 중심 문제는 조금 다르다.

```text
현재 state + props
→ 지금 화면이 어떻게 보여야 하는가?
```

이를 간단히 표현하면 다음과 같다.

```text
UI = f(state)
```

```tsx
function UserProfile({ user }: { user: User }) {
  return <h1>{user.name}</h1>;
}
```

이 component는 장수하는 객체에게 DOM 변경 명령을 보내는 모델보다, 입력이 주어졌을 때 어떤 UI가 되어야 하는지를 선언하는 함수에 가깝다. React가 함수 component, 단방향 data flow와 선언적 rendering을 중심에 두는 이유도 이 문제 형태와 연결된다.

## UI는 분류 계층보다 조립 트리에 가깝다

OOP의 상속은 “A는 B의 한 종류다”라는 관계를 표현할 때 자연스럽다.

```text
PaymentMethod
├── CardPayment
├── BankTransfer
└── MobilePayment
```

반면 UI는 “무엇 안에 무엇을 배치하는가?”라는 트리 구조가 더 중요하다.

```text
Page
└── Dialog
    ├── Header
    ├── Form
    └── Actions
```

그래서 `PaymentDialog extends Dialog` 같은 상속보다 다음 합성이 화면별 예외와 조합을 다루기 쉽다.

```tsx
<Dialog
  title="결제"
  footer={<PaymentActions />}
>
  <PaymentForm />
</Dialog>
```

화면 하나만 다른 안내문이나 footer를 가져야 할 때 상속 hierarchy를 바꾸지 않고 조립만 변경할 수 있다. React의 `children`, slot, compound component와 headless component는 이 특성을 활용한다.

## React는 OOP의 목적을 다른 구조로 표현한다

| 설계 개념 | Java·Spring의 대표 표현 | React·TypeScript의 대표 표현 |
|---|---|---|
| 캡슐화 | class와 private member | component, module, custom hook |
| 추상화 | interface | props type, 함수 type, hook API |
| 다형성 | interface 구현체 교체 | component나 함수를 값으로 전달 |
| 의존성 주입 | constructor injection | props, Context, factory |
| OCP | 새 구현체와 configuration 교체 | composition, slot, registry, adapter |
| SRP | Controller·Service·Repository 분리 | component·hook·service boundary |
| 소유권 | 객체 생성과 Bean lifecycle | state ownership과 component lifecycle |

React에서 값을 외부에서 전달하는 것은 그 자체로 dependency injection이다.

```tsx
<UserPage repository={httpUserRepository} />
```

Test에서는 다른 구현을 전달할 수 있다.

```tsx
<UserPage repository={fakeUserRepository} />
```

Spring Container처럼 framework가 type을 찾아 자동으로 연결하지 않을 뿐, 상위 조립부가 구체 구현을 선택하고 하위 코드가 계약에 의존한다는 원리는 같다.

## 함수형 접근에도 소프트웨어 설계가 있다

함수 component를 단순히 “함수 몇 개를 작성하는 방식”으로 보면 React가 구조를 포기한 것처럼 보일 수 있다. 그러나 함수형·선언형 설계도 다음 요소를 다룬다.

- module과 component boundary
- 명확한 입력과 출력
- 변경을 추적하기 쉬운 data flow
- side effect의 격리
- 함수와 component 합성
- dependency 전달
- 안정적인 public API
- state ownership

Java·Spring의 중요한 질문이 “이 객체를 누가 생성하고 어떤 구현을 주입하는가?”라면 React의 중요한 질문은 “이 state를 누가 소유하고 어떤 component에 전달하는가?”다. 둘 다 책임과 소유권, 의존성과 변경 전파를 관리한다.

## 한 시스템에서는 OOP와 React 방식이 연결된다

실제 Full-stack application은 하나의 패러다임으로만 구성되지 않는다.

```text
React component
→ custom hook
→ application use case
→ repository contract
→ HTTP adapter
→ Spring Controller
→ Service
→ Repository
```

UI에 가까운 부분에서는 선언형 rendering, 함수와 합성이 강하고, domain rule과 외부 기술 경계에서는 객체 협력, interface와 DIP가 더 선명할 수 있다. 중요한 것은 모든 계층을 같은 문법으로 통일하는 것이 아니라 각 문제에 맞는 도구를 쓰면서 dependency 방향과 변경 범위를 설명할 수 있는가이다.

## Spring과 React를 함께 공부하는 관점

Spring에서는 다음 질문을 훈련한다.

- 객체와 Bean을 누가 생성하고 소유하는가?
- 어떤 interface에 어떤 구현이 주입되는가?
- Request와 transaction은 어느 boundary를 통과하는가?
- 저장 기술 변경이 application logic에 어디까지 전파되는가?

React에서는 다음 질문을 훈련한다.

- State를 누가 소유하는가?
- Render와 event handler는 어떻게 연결되는가?
- Derived state와 effect를 어떻게 구분하는가?
- Component boundary와 public props는 안정적인가?
- 변경이 component tree 어디까지 전파되는가?

표면 문법은 다르지만 공통 질문은 같다.

> 변경이 발생하면 어디까지 수정되고, 누가 상태와 의존성을 책임지는가?

이 관점을 사용하면 OOP와 React를 경쟁 관계로 보지 않고 서로 다른 문제를 해결하는 도구로 연결할 수 있다.

## Spring 원칙을 Frontend 언어로 번역하기

Spring 예제에서는 Service가 Memory·JDBC·JPA 같은 구체 저장소를 직접 선택하지 않는다. Service는 Repository 계약에 의존하고, configuration이 실제 구현을 조립한다. 저장 방식을 바꿀 때 business code 대신 조립부만 변경한다.

```text
Spring
MemberService → MemberRepository ← Memory/JDBC/JPA

Frontend
화면·use case → UserRepository ← HTTP/Mock/Storage 구현
```

Frontend에서도 원리는 같다.

- 안정적인 화면·business rule은 구체적인 외부 기술을 직접 알지 않는다.
- 네트워크·저장소·브라우저 API 같은 변경 지점은 작은 계약 뒤에 둔다.
- 실제 구현 선택은 app 진입점, feature 조립부, props 또는 Context에서 한다.

OCP는 “기존 파일을 영원히 수정하지 않는다”는 뜻이 아니다. 새 요구사항이 들어올 때 핵심 정책 여러 곳을 고치는 대신 새 구현을 추가하고 조립 지점의 변경으로 범위를 국소화하는 설계 원칙이다.

## 가장 먼저 적용할 곳: 외부 경계

Frontend에서 DIP의 효과가 가장 분명한 곳은 application과 바깥 환경이 만나는 경계다.

- REST·GraphQL·BFF 같은 network transport
- `localStorage`, cookie, IndexedDB 같은 저장소
- analytics와 error reporting SDK
- feature flag provider
- browser API와 native bridge
- 운영 API와 test·개발용 fake 구현

TypeScript에서는 class hierarchy 없이 type과 객체만으로 계약을 만들 수 있다.

```ts
export type UserRepository = {
  getUser(id: string): Promise<User>;
  updateUser(id: string, data: Partial<User>): Promise<User>;
};

export const httpUserRepository: UserRepository = {
  async getUser(id) {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  },

  async updateUser(id, data) {
    const response = await fetch(`/api/users/${id}`, {
      method: "PATCH",
      body: JSON.stringify(data),
    });
    return response.json();
  },
};
```

테스트나 Backend 준비 전 개발에서는 같은 계약의 fake 구현을 사용할 수 있다.

```ts
export const fakeUserRepository: UserRepository = {
  async getUser(id) {
    return { id, name: "테스트 사용자" };
  },

  async updateUser(id, data) {
    return { id, name: data.name ?? "테스트 사용자" };
  },
};
```

화면이나 use case는 `fetch`가 아니라 계약을 받는다.

```ts
export function createLoadUser(repository: UserRepository) {
  return (id: string) => repository.getUser(id);
}
```

이 구조의 목적은 “Repository pattern을 사용했다”는 모양이 아니다. HTTP 변경, test double, 아직 완성되지 않은 API 등 실제 교체 요구가 생겼을 때 화면과 business rule의 수정 범위를 줄이는 것이다.

## React의 DI는 props·Context·module 조립이다

React에서는 전용 DI container가 없어도 값을 외부에서 전달하는 것 자체가 dependency injection이다.

```tsx
function UserPage({
  repository,
  userId,
}: {
  repository: UserRepository;
  userId: string;
}) {
  const userQuery = useQuery({
    queryKey: ["user", userId],
    queryFn: () => repository.getUser(userId),
  });

  // loading·error·success UI
}
```

Application의 조립 지점에서 운영 구현을 넣고, test에서는 fake를 넣을 수 있다.

```tsx
<UserPage
  repository={httpUserRepository}
  userId={userId}
/>
```

의존성을 여러 단계의 component가 공유한다면 Context Provider를 조립 지점으로 사용할 수 있다. 반대로 단순한 feature에 무조건 Context나 DI library를 도입하면 의존 경로만 숨길 수 있으므로, 가까운 관계에서는 props나 함수 인자가 더 명확하다.

Angular처럼 DI가 framework 실행 모델에 포함된 환경은 Spring과 유사한 형태가 자연스럽다. React에서는 module import, props, Context와 factory function만으로 충분한 경우가 많다.

## UI 레이어의 OCP는 합성으로 표현된다

UI에서는 interface hierarchy보다 확장 지점을 열어 놓는 component API가 중요하다. 내부에서 모든 variant를 판별하면 새 종류가 추가될 때 공통 component를 계속 수정하게 된다.

```tsx
function Dialog({ type }: { type: DialogType }) {
  if (type === "confirm") return <ConfirmDialog />;
  if (type === "payment") return <PaymentDialog />;
  if (type === "warning") return <WarningDialog />;
}
```

안정적인 구조만 제공하고 변하는 표현은 호출자가 조립하게 만들 수 있다.

```tsx
function Dialog({
  title,
  children,
  footer,
}: {
  title: string;
  children: React.ReactNode;
  footer: React.ReactNode;
}) {
  return (
    <section role="dialog">
      <header>{title}</header>
      <main>{children}</main>
      <footer>{footer}</footer>
    </section>
  );
}
```

```tsx
<Dialog
  title="결제"
  footer={<PaymentActions />}
>
  <PaymentForm />
</Dialog>
```

`children`, slot props, render props, compound component와 headless component는 모두 이런 합성의 변형이다. 공통 component는 focus 관리나 keyboard interaction 같은 안정적인 행위를 제공하고, 구체적인 내용과 표현은 사용하는 쪽에서 조립할 수 있다.

다만 모든 `variant`가 잘못된 것은 아니다. 제한되고 안정적인 design token 집합을 표현하는 `size="sm"`이나 `tone="danger"`는 유용하다. 변화가 무한히 열려야 하는 부분까지 enum 형태의 variant로 중앙집중화할 때 문제가 커진다.

## 조건 분기를 전략 또는 레지스트리로 분리하기

종류별 행위나 렌더링이 커지면 하나의 `switch` 안에 모두 넣기보다 각 구현을 독립시키고 map에서 선택할 수 있다.

```tsx
const NOTIFICATION_RENDERERS: Record<
  NotificationType,
  React.ComponentType<NotificationProps>
> = {
  comment: CommentNotification,
  mention: MentionNotification,
};

function NotificationItem(props: NotificationProps) {
  const Renderer = NOTIFICATION_RENDERERS[props.notification.type];
  return <Renderer {...props} />;
}
```

새 notification 종류를 추가할 때 새 component와 registry 항목을 추가한다. Registry 자체는 수정되지만 공통 렌더링 흐름과 기존 구현은 건드리지 않는다. 즉, OCP의 실무적 효과는 변경을 완전히 없애는 것이 아니라 예측 가능한 조립 지점으로 모으는 것이다.

행위 하나만 교체한다면 interface보다 함수 타입이 더 간단하다.

```ts
type PricePolicy = (order: Order) => Money;

function calculateTotal(
  order: Order,
  pricePolicy: PricePolicy,
) {
  return pricePolicy(order);
}
```

Java에서 단일 method interface가 담당하던 역할을 JavaScript·TypeScript에서는 일급 함수가 자연스럽게 맡는다.

## TypeScript에서 Spring 모양을 복사하지 않는 이유

### 구조적 타이핑

Java는 `implements`로 구현 관계를 명시하지만 TypeScript는 필요한 구조를 만족하면 같은 계약으로 취급한다. `UserRepositoryImpl` class를 만들지 않고 type과 객체 literal만으로도 충분할 수 있다.

### 일급 함수

교체할 행위가 하나라면 interface와 class 대신 `(input) => output` 형태의 함수가 더 작은 계약이다. callback, event handler와 selector도 이미 이 방식을 사용한다.

### 조립 방식

Spring Container가 Bean을 찾아 연결하는 대신 React에서는 import, props, Context와 factory가 조립부가 된다. 전용 DI container는 object graph가 복잡하고 실제 교체 요구가 있을 때 검토해도 늦지 않다.

### Browser 비용

TypeScript의 type과 interface 자체는 compile 결과에서 제거되므로 bundle 비용을 만들지 않는다. 하지만 runtime DI container, decorator metadata와 범용 abstraction library를 추가하면 browser에 전달할 code와 초기화 복잡성이 늘 수 있다. 따라서 “추상화는 무조건 비싸다”가 아니라 runtime 구조가 실제 이익을 주는지 판단해야 한다.

## 추상화가 실패하는 방식

Frontend의 UI 요구사항은 화면별 예외, interaction과 responsive 차이를 자주 만든다. 실제 사례를 보기 전에 공통 API를 설계하면 다음 문제가 생길 수 있다.

- 하나뿐인 구현을 위해 interface와 `Impl`을 쌍으로 만든다.
- 서로 의미가 다른 UI를 외형이 비슷하다는 이유로 합친다.
- 범용 component의 props가 boolean과 예외 flag로 계속 늘어난다.
- React component에 Controller–Service–Repository 계층을 기계적으로 대응시킨다.
- 추상화가 제거하려던 조건문이 추상화 내부로 이동할 뿐이다.

Wiki의 학습 원칙도 모든 class에 interface를 만드는 것을 DIP로 오해하지 말고, 구현·변경·debug 경험으로 추상화의 필요성을 판단하라고 강조한다. 공통 component 역시 실제 중복을 관찰하고 의미와 동작이 같은지 확인한 뒤 추출하는 편이 안전하다.

## 추상화 여부를 결정하는 질문

다음 질문에 구체적으로 답할 수 있을 때 추상화의 가치가 높다.

1. 어떤 구현이 실제로 교체되는가?
2. 교체할 때 보호하고 싶은 핵심 정책은 무엇인가?
3. 운영 구현 외에 fake·mock·다른 platform 구현이 필요한가?
4. 반복된 두 사례에서 공통점과 차이점을 이미 확인했는가?
5. 계약이 구현 세부사항보다 안정적인가?
6. 추상화 후 변경이 한 경계나 조립 지점으로 줄어드는가?

반대로 “나중에 필요할지도 모른다” 외에는 답이 없다면 구체 구현으로 시작하는 편이 낫다. 두 번째 구현이나 실제 중복이 나타났을 때 공통점을 추출해도 된다. API·browser boundary처럼 test double과 운영 구현이 처음부터 필요한 곳은 일찍 분리할 근거가 충분하다.

## 제품 단계에 따른 적용 강도

짧은 수명의 prototype이나 빠른 검증 단계에서는 추상화 비용이 얻는 이익보다 클 수 있다. 반면 오래 유지되는 제품, 여러 실행 환경, 복수의 data source, 반복되는 test double과 여러 팀이 공유하는 UI에서는 안정적인 계약과 조립 경계의 가치가 커진다.

따라서 “Frontend에도 Clean Architecture를 적용할 것인가”를 일괄적으로 결정하기보다 feature 수명, 변경 빈도, team ownership, testability와 migration 비용을 기준으로 경계를 선택한다. Architecture는 folder 수가 아니라 변경이 전파되는 방향을 설명해야 한다.

## 추천 실습 순서

가장 먼저 API 호출부 하나에 적용한다.

```text
1. Component나 hook 안의 직접 fetch를 찾는다.
2. 화면이 실제로 필요한 operation만 작은 type으로 정의한다.
3. 기존 fetch code를 HTTP 구현으로 옮긴다.
4. Props·factory 또는 Context에서 구현을 주입한다.
5. Fake 구현으로 화면의 loading·error·success behavior를 검증한다.
6. 수정 전후 dependency와 test 난이도를 비교한다.
```

그다음 실제로 반복된 Dialog나 page shell에서 합성을 연습하고, 종류별 조건문이 커질 때 renderer registry나 함수 전략을 적용한다. 추상화의 이름보다 변경이 국소화되고 test가 쉬워졌는지를 성공 기준으로 삼는다.

## 최종 판단

Java와 Spring에서 배운 OCP·DIP는 Frontend에서도 유효하다. 가져와야 하는 것은 `interface + Impl + container`라는 문법이 아니라 다음 질문이다.

> 무엇이 안정적인 정책이고, 무엇이 자주 바뀌는 구현이며, 그 변경을 어디에서 조립할 것인가?

Frontend에서는 그 답이 주로 type·함수, adapter, props·Context, component composition과 registry로 나타난다. 실제 변화가 있는 경계부터 작게 적용하고, UI에는 두 번째 사례를 본 뒤 추상화하는 것이 가장 안전한 출발점이다.

## See Also

- [Frontend에서 OCP와 의존성 역전을 적용하는 실무 가이드](frontend-ocp-and-dependency-inversion-practical-guide-2026-08-24.md)
- [Spring Bean과 의존관계 설정](../spring/spring-beans-and-dependency-injection.md)
- [AI 시대의 Software Engineering 학습 전략](ai-era-software-engineering-learning-strategy-2026-08-23.md)
- [Superpowers 기반 Brownfield 연습 워크플로 초기 설계](../ai-agents/superpowers-brownfield-practice-workflow-initial-design.md)
- [AI 중심 실무 환경의 Full-stack 개발자 6개월 학습 로드맵](ai-native-fullstack-learning-roadmap-2026-08-23.md)
