# Frontend에서 OCP와 의존성 역전을 적용하는 실무 가이드

> Sources: [Spring Bean과 의존관계 설정](../spring/spring-beans-and-dependency-injection.md); [AI 시대의 Software Engineering 학습 전략](ai-era-software-engineering-learning-strategy-2026-08-23.md); [Superpowers 기반 Brownfield 연습 워크플로 초기 설계](../ai-agents/superpowers-brownfield-practice-workflow-initial-design.md); [AI 중심 실무 환경의 Full-stack 개발자 6개월 학습 로드맵](ai-native-fullstack-learning-roadmap-2026-08-23.md)
> Archived: 2026-08-24

## Overview

OCP(Open–Closed Principle)와 인터페이스·구현 분리는 Frontend에도 적용할 수 있고 실제 제품 코드에서도 사용된다. 다만 Spring의 `interface + Impl class + DI container` 구조를 그대로 복제하기보다는, 변화하는 경계를 TypeScript type과 함수로 감싸고 React의 props·Context·합성(composition)으로 구현을 주입하는 편이 자연스럽다. 핵심은 모든 코드에 추상화를 추가하는 것이 아니라, 다음 변경에서 보호할 안정적인 정책과 교체할 구현을 구분하는 것이다.

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

- [Spring Bean과 의존관계 설정](../spring/spring-beans-and-dependency-injection.md)
- [AI 시대의 Software Engineering 학습 전략](ai-era-software-engineering-learning-strategy-2026-08-23.md)
- [Superpowers 기반 Brownfield 연습 워크플로 초기 설계](../ai-agents/superpowers-brownfield-practice-workflow-initial-design.md)
- [AI 중심 실무 환경의 Full-stack 개발자 6개월 학습 로드맵](ai-native-fullstack-learning-roadmap-2026-08-23.md)
