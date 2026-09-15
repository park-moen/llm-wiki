# Kotlin JPA 공식 문서 companion

> Source: https://kotlinlang.org/docs/no-arg-plugin.html; https://kotlinlang.org/docs/all-open-plugin.html; https://docs.spring.io/spring-boot/reference/features/kotlin.html
> Collected: 2026-08-22
> Published: Unknown

Kotlin 공식 no-arg compiler plugin 문서는 annotation이 붙은 class에 synthetic zero-argument constructor를 생성하며, reflection을 사용하는 JPA가 이를 이용해 class instance를 만들 수 있다고 설명한다. `kotlin-jpa` plugin은 JPA annotation을 대상으로 no-arg 구성을 제공한다.

Kotlin class와 member는 기본적으로 `final`이다. All-open compiler plugin은 지정한 annotation이 붙은 class와 member를 `open`으로 처리한다. Kotlin 공식 문서는 framework proxy를 위해 이 기능이 필요할 수 있다고 설명하며, Spring annotation에는 `kotlin-spring` plugin을 사용할 수 있다.

Spring Boot Kotlin 공식 문서는 Spring이 proxy를 만들 수 있도록 `kotlin-spring` plugin을 구성할 것을 권장한다. 또한 Kotlin의 null safety를 사용하면 Java의 `Optional` wrapper 대신 nullable type으로 값의 부재를 표현할 수 있다고 설명한다.

JPA entity는 Spring Bean과 역할이 다르므로 `kotlin-spring`만으로 entity의 JPA 요구사항이 모두 해결된다고 가정하지 않는다. Project가 사용하는 Kotlin compiler version에 맞춰 `kotlin-jpa`의 no-arg 및 entity open 지원 범위를 확인한다.

