# Spring Boot Kotlin Multi Module로 구성해보는 헥사고날 아키텍처

> Source: https://techblog.woowahan.com/12720/
> Collected: 2026-10-23
> Published: 2023-07-11

## 들어가는 말

백엔드 개발자들은 프로젝트를 처음 구성할 때 사용할 기술에 관해 많은 고민을 합니다.

우리 데이터베이스(DB)로 CockroachDB가 요즘 흥한다던데 이걸 사용해 보는 건 어떨까?

HTTP 클라이언트로 Retrofit이 좋을까 Feign을 사용하는 것이 좋을까?

MySQL은 좀 진부하지 않아? 새로운 걸 좀 해볼까?

이 과정은 짧게는 개발자들 사이의 갑론을박에서 길게는 윗선의 결정까지 적지 않은 시간적 비용이 소모됩니다.

문제는 이렇게 어려운 선택들이 프로젝트를 성공적으로 이끌게 되었는가? 물론 그럴 때도 있겠지만 프로젝트 중후반에서야  "아.. 이거 OO를 사용하면 안 되는 거였는데.."라는 후회(?)를 하게 되는 경우가 적지 않게 있고, 이미 선정된 기술들이 이것 저곳 비즈니스 코드들 깊이 뿌리내려 있는 상황이라 이러지도 저러지도 못하는 상황에서 힘들게 프로젝트를 마무리하게 되는 것을 심심치 않게 보았습니다.

이것은 비단 백엔드 서버 개발만의 문제가 아닌 다양한 기술적 선택지가 있는 프런트엔드도 마찬가지일 거라고 생각합니다.

## 배경

헥사고날 아키텍처는 비즈니스 요구사항을 빠르게 개발할 때 기술 선택에 대한 고민으로 소모되는 비용을 아낄 수 있는 대표적인 애플리케이션 아키텍처입니다.

오늘은 헥사고날 아키텍처를 도입하면서 겪었던 경험들을 기록하여 초기 프로젝트 세팅 시 헥사고날 아키텍처를 고민하고 있거나, 새로운 아키텍처 도입이 망설여져 도전하지 못하는 백엔드 개발자분들께 작게나마 도움이 되었으면 합니다.

이하 내용은 Spring Boot Kotlin Multi Moudule로 핵사고날 아키텍처의 이론과 개념, "[DDD, Hexagonal, Onion, Clean, CQRS, … How I put it all together](https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/)"를 실제로 구현한 사례입니다.

헥사고날 아키텍처는 기술에 독립적이어야 하는데 Spring이라니?라고 의아하실 수 있지만, 가장 대중적으로 많이 사용하고 있는 Framework이며, 100% 이론을 실제화하기보단 상황에 맞는 방법을 택하여 생산성을 높이는 선택을 하는 것이 맞을 것 같아 약간 배타적일 수 있지만 부득이하게 Spring에 종속적인 모듈 구성을 하게 되었습니다.

🗒️ 헥사고날 아키텍처에 대한 이론적인 설명은 ChatGPT, Google 검색을 통해 충분히 확인할 수 있을 것 같아 따로 언급하지 않겠습니다. 우아콘2022에서 소프트웨어 아키텍처를 알기 쉽게 설명한 김현수 님의 발표 영상, “[기획자님들! 개발자가 아키텍처에 집착하는 이유, 쉽게 알려드립니다](https://www.youtube.com/watch?v=saxHxoUeeSw&list=PLgXGHBqgT2TsFnKRe3_kvFXDFUWxaUvQ2&index=7)“도 미리 확인하시는 것을 추천합니다.

## 구성

헥사고날 아키텍처를 도입하기로 한 프로젝트 ceo-united는 배달의민족에서 사장님들이 사용하는 포스(POS) 프로그램의 백엔드를 기능을 담당하기 위한 프로젝트로 총 4개의 핵사곤(Layer)으로 정의하였습니다.

- Domain Hexagon

- Application Hexagon

- Framework Hexagon

- Bootstrap Hexagon

각 헥사곤별 구성과 구현 예시를 자세히 살펴보겠습니다.

![](https://techblog.woowahan.com/wp-content/uploads/2023/06/100-explicit-architecture-svg-1024x700.jpg)
[그림 1] 출처: [https://herbertograca.com/tag/explicit-architecture](https://herbertograca.com/tag/explicit-architecture)

### Domain Hexagon

- [그림 1] Domain Layer 영역

- DDD(도메인 주도 개발)의 그 Domain Layer로 기술에 독립적인 POJO로 개발

- Utility, Extension, Constants, Entity, VO(Enum 포함), Aggregate를 포함하며 프로젝트 도메인의 비즈니스 룰을 정의

- POJO로 구현하기 때문에 Spring의 Component, Service annotation 등 비사용

- 비즈니스 규칙을 위한 Service 개념이 존재할 수 있어야 하지만 static class의 메서드로만 정의

- 실제로 gradle에 어떤 dependency도 포함하지 않음

build.gradle.kts

```
import org.springframework.boot.gradle.tasks.bundling.BootJar

val jar: Jar by tasks
val bootJar: BootJar by tasks

bootJar.enabled = false
jar.enabled = true

dependencies {

}

```

![](https://techblog.woowahan.com/wp-content/uploads/2023/07/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2023-06-27-%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB-7.42.47-600x288-1.png)

### Application Hexagon

- [그림 1] Application Layer 영역

- Domain의 구성요소를 사용하여 시스템이 가지는 기능/사례(usecase)를 정의한 집합

- logging, exception, usecase, outputPort(interface) 등을 포함

- DB가 어떤 것인지, 외부에서 시스템을 가동하기 위한 기술은 무엇인지 아무것도 알 필요가 없다. 알아서도 안된다

- 팀에 신규 인력이 추가되거나 더 나아가 PO(ProductOwner), PM(ProductManager) 같이 비 개발 인력도 읽을 수 있을만한 쉬운 코드로 작성

- POJO로 개발하고 다른 영역에서 DI를 제어해야 하지만 "Spring framework을 버리고 다른 DI Framework를 사용할 여지가 있는가?"라는 물음에 "그렇지 않다"라는 결론을 내렸고 각 기능(usecase) 들에 대하여 Service annotation을 사용한 Service로 정의

- 의존성은 Domain Hexagon에 대해서만 가짐

build.gradle.kts

```
import org.springframework.boot.gradle.tasks.bundling.BootJar

val jar: Jar by tasks
val bootJar: BootJar by tasks

bootJar.enabled = false
jar.enabled = true

dependencies {
    implementation(project(":ceo-united-domain"))

    implementation("org.springframework.data:spring-data-commons")
}

```

AppConfig.kt

(component scan의 lazyInit option에 대한 내용은 하단에 기재합니다)

```
@Configuration
@ComponentScan(basePackages = ["com.baemin.ceo.united.application.usecase"], lazyInit = true)
class AppConfig
```

![](https://techblog.woowahan.com/wp-content/uploads/2023/07/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2023-06-27-%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB-7.44.31-600x244-1.png)

### Framework Hexagon

- [그림 1] Secondary/Driven Adapters (Infrastructure) 영역

- Application hexagon이 소유한 outputPort (interface) 구현체들의 집합

- Framework Hexagon은 여러 개의 module로 분리, 관리

- Application Hexagon과 마찬가지로 Spring에 대한 의존성은 가짐

- 각 Framework Hexagon은 각 기술의 이름을 딴 config class를 포함하며, config class는 그 기술이 사용할 component들을 bean으로 등록할 수 있게 scan 영역을 격리

#### ceo-united-framework-aws

build.gradle.kts

```
import org.springframework.boot.gradle.tasks.bundling.BootJar

val jar: Jar by tasks
val bootJar: BootJar by tasks

bootJar.enabled = false
jar.enabled = true

dependencies {
    implementation(project(":ceo-united-domain"))
    implementation(project(":ceo-united-application"))

    implementation("org.springframework.boot:spring-boot-starter-data-mongodb")
    implementation("software.amazon.awssdk:s3")
    implementation("software.amazon.awssdk:netty-nio-client")
}
```

MongoConfig.kt

```
@Configuration
@EnableMongoRepositories
@ComponentScan(basePackages = ["com.baemin.ceo.united.framework.aws.adapter.mongo"])
class MongoConfig {
```

S3Config.kt

```
@Configuration
@ComponentScan(basePackages = ["com.baemin.ceo.united.framework.aws.adapter.s3"])
class S3Config {
```

![](https://techblog.woowahan.com/wp-content/uploads/2023/07/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2023-06-27-%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB-7.45.57-600x326-1.png)

#### ceo-united-framework-retrofit

build.gradle.kts

```
import org.springframework.boot.gradle.tasks.bundling.BootJar

val jar: Jar by tasks
val bootJar: BootJar by tasks

bootJar.enabled = false
jar.enabled = true

dependencies {
    implementation(project(":ceo-united-domain"))
    implementation(project(":ceo-united-application"))
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-jackson:2.9.0")
    implementation("com.squareup.okhttp3:okhttp:3.14.9")
    implementation("com.squareup.okhttp3:logging-interceptor:3.14.9")
```

RetrofitConfig.kt

```
@Configuration
@ComponentScan(
    basePackages = [
        "com.baemin.ceo.united.framework.retrofit.adapter",
        "com.baemin.ceo.united.framework.retrofit.config"
    ]
)
class RetrofitConfig {
```

![](https://techblog.woowahan.com/wp-content/uploads/2023/07/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2023-06-27-%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB-7.46.43-600x248-1.png)

### Bootstrap Hexagon

- [그림 1] Primary/Driving Adapters (User Interface) 영역

- 프로그램의 기능을 사용하기 위한 시작점

- 우리는 RESTAPI를 제공하는 Webservice, Apache Kafka, AWS의 SQS(Simple Queue Service)를 구독하는 consumer가 존재

- Domain, Application, Framework Hexagon에 대하여 의존적, 애플리케이션 구동에 필요한 모듈의 config만 선별적으로 import

#### ceo-united-bootstrap-api

build.gradle.kts

```
apply(plugin = "kotlin-kapt")

dependencies {
    implementation(project(":ceo-united-domain"))
    implementation(project(":ceo-united-application"))
    implementation(project(":ceo-united-framework-aws"))
    implementation(project(":ceo-united-framework-retrofit"))

    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("com.fasterxml.jackson.datatype:jackson-datatype-jdk8")
    implementation("com.fasterxml.jackson.datatype:jackson-datatype-jsr310")
    implementation("org.springframework.boot:spring-boot-starter-webflux")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    implementation("org.springframework.boot:spring-boot-starter-aop")
```

CeoUnitedApiApplication.kt

```
@SpringBootApplication
@Import(
    value = [AppConfig::class, MongoConfig::class, RetrofitConfig::class]
)
@ConfigurationPropertiesScan
class CeoUnitedApiApplication

fun main(args: Array<String>) {
    runApplication<CeoUnitedApiApplication>(*args)
}

```

![](https://techblog.woowahan.com/wp-content/uploads/2023/07/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA-2023-06-27-%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB-7.47.51-600x269-1.png)

#### ceo-united-bootstrap-worker

build.gradle.kts

```
apply(plugin = "kotlin-kapt")

dependencies {
    implementation(project(":ceo-united-domain"))
    implementation(project(":ceo-united-application"))
    implementation(project(":ceo-united-framework-aws"))
    implementation(project(":ceo-united-framework-cache"))
    implementation(project(":ceo-united-framework-retrofit"))

    kapt("org.springframework.boot:spring-boot-configuration-processor")

    implementation("org.springframework.cloud:spring-cloud-starter-aws-messaging:2.2.6.RELEASE")
    implementation("org.springframework.kafka:spring-kafka")
```

CeoUnitedWorkerApplication.kt

```
@SpringBootApplication(
    scanBasePackageClasses = [
        CeoUnitedWorkerApplication::class
    ]
)
@Import(value = [AppConfig::class, MongoConfig::class, RetrofitConfig::class, CacheConfig::class, S3Config::class])
@ConfigurationPropertiesScan
class CeoUnitedWorkerApplication
fun main(args: Array<String>) {
    runApplication<CeoUnitedWorkerApplication>(*args)
}
```

## 개발사례

사장님이 사용하는 포스(POS)가 시작될 때 클라이언트 환경설정을 조회하되, 그 설정이 존재하지 않으면 초기화된 설정으로 제공하고 싶다.

위와 같은 요구사항이 발생하였을 때 우리가 구성한 아키텍처에서는 다음과 같이 개발됩니다.

(code snippet은 콘셉트를 알기 위함으로 일부 발췌된 코드입니다.)

![](https://techblog.woowahan.com/wp-content/uploads/2023/07/diagram-1-1024x447.png)

-

Domain Hexagon에 필요한 Entity를 정의

MerchantSetting.kt

```
data class MerchantSetting(
val merchantNo: String,
val checkForUpdates: Boolean,
val notifySoundVolumn: Int,
) {
companion object {
    fun getDefault(merchantNo: String) = MerchantSetting(merchantNo, false, 10)
}
}
```

-

Application Hexagon에 usecase를 정의

MerchantSettingRetrieveUseCase.kt

```
interface MerchantSettingRetrieveUseCase {
fun getOrCreateDefault(merchantNo: String): MerchantSetting
}
```

-

Application Hexagon에 outputPort interface를 생성

MerchantSettingManagementOutputPort.kt

```
interface MerchantSettingManagementOutputPort {
fun getMerchantSetting(merchantNo: String): MerchantSetting?
fun createMerchantSetting(merchantSetting: MerchantSetting): MerchantSetting
}
```

-

Application Hexagon에 usecase에 대한 구현체(inputPort) 개발

MerchantSettingRetrieveInputPort.kt

```
@Service
class MerchantSettingRetrieveInputPort @Autowired constructor(
private val merchantSettingManagementOutputPort: MerchantSettingManagementOutputPort
): MerchantSettingRetrieveUseCase {
override fun getOrCreateDefault(merchantNo: String): MerchantSetting {
    return merchantSettingManagementOutputPort.getMerchantSetting(merchantNo) ?:
        merchantSettingManagementOutputPort.createMerchantSetting(MerchantSetting.getDefault(merchantNo))
}
}
```

-

usecase에 대한 기능 테스트, 검증

MerchantSettingRetrieveInputPortTest.kt

```
class MerchantSettingRetrieveInputPortTest : FreeSpec({
val merchantSettingManagementOutputPort = mockk<MerchantSettingManagementOutputPort>(relaxed = true)
val sut = MerchantSettingRetrieveInputPort(merchantSettingManagementOutputPort)

"getOrCreateDefault" - {
    "저장되어 있는 설정이 있으면 생성하지 않고 응답" {
        every { merchantSettingManagementOutputPort.getMerchantSetting(any()) } returns mockk()

        sut.getOrCreateDefault("1")

        verify(exactly = 1) { merchantSettingManagementOutputPort.getMerchantSetting(any()) }
        verify(exactly = 0) { merchantSettingManagementOutputPort.createMerchantSetting(any()) }
    }
    "저장되어 있는 설정이 없으면 새로 생성하여 응답" {
        every { merchantSettingManagementOutputPort.getMerchantSetting(any()) } returns null
        every { merchantSettingManagementOutputPort.createMerchantSetting(any()) } returns mockk()

        sut.getOrCreateDefault("1")

        verify(exactly = 0) { merchantSettingManagementOutputPort.getMerchantSetting(any()) }
        verify(exactly = 1) { merchantSettingManagementOutputPort.createMerchantSetting(any()) }
    }
})
```

-

Framework-hexagon(ceo-united-framework-aws)에 outputPort의 구현체(adapter) 개발

MerchantSettingAdapter.kt

```
@Component
class MerchantSettingAdapter @Autowired constructor (mongoTemplate: MongoTemplate): MerchantSettingManagementOutputPort {
override fun getMerchantSetting(merchantNo: String): MerchantSetting? {
    return mongoTemplate.find(
        Query.query(MerchantSettingMongo::merchantNo isEquals merchantNo),
        MerchantSettingMongo::java.class
    )?.toDomain()
}
override fun createMerchantSetting(merchantSetting: MerchantSetting): MerchantSetting {
    return mongoTemplate.save(MerchantSettingMongo.from(merchantSetting))
}
}
```

MerchantSettingMongo.kt

```
@Document("merchant_setting")
data class MerchantSettingMongo(
@Id
val id: String?,
@Field
val merchantNo: String,
@Field
 val checkForUpdates: Boolean,
 @Field
val notifySoundVolumn: Int,
) {
fun toDomain(): MerchantSetting {
    TODO("")
}

companion object {
    fun from(merchantSetting: MerchantSetting): MerchantSettingMongo {
        TODO("")
    }
}
}
```

-

adatper에 대한 기능 테스트, 검증

MerchantSettingAdapterTest.kt

```
@IntegrationTestSupport
class MerchantSettingAdapterTest(
@Autowired private val merchantSettingAdapter: MerchantSettingAdapter,
) : MongoIntegrationTest() {
@Test
@DisplayName("존재하는 설정을 가져온다")
@MongoTest(["com/baemin/ceo/united/framework/aws/adapter/mongo/MerchantSetting.json"])
fun getExistMerchantSetting() {
    val actual = merchantSettingAdapter.getMerchantSetting("exist_user")
    requireNotNull(actual)
}

@Test
@DisplayName("설정이 없으면 null을 응답한다")
@MongoTest(["com/baemin/ceo/united/framework/aws/adapter/mongo/MerchantSetting.json"])
fun returnNullWhenNotExists() {
    val actual = merchantSettingAdapter.getMerchantSetting("not_exist_user")
    actual == null
}
}
```

-

bootstrap hexagon (ceo-united-bootstrap-api)에 usecase를 이용한 서비스 노출

MerchantSettingController.kt

```
@RestController
class MerchantSettingController @Autowired constructor (
private val merchantSettingRetrieveUseCase: MerchantSettingRetrieveUseCase
) {
@GetMapping("/{merchantNo}/setting")
fun getSetting(merchantNo: String) {
    return MerchantSettingResponse.from(merchantSettingRetrieveUseCase.getOrCreateDefault(merchantNo))
}
}
data class MerchantSettingResponse(
val checkForUpdates: Boolean,
val notifySoundVolumn: Int,
) {
companion object {
    fun from(merchantSetting: MerchantSetting) {
        TODO("impl")
    }
}
}
```

-

통합테스트, 검증

## 고민했던 부분

### 헥사곤 간의 객체변환

예시에서 볼 수 있듯 각 포트로의 데이터 교환에 있어서 헥사곤 영역에 맞는 클래스로 필드 매핑이 계속 발생합니다.

(도메인에서 정의한 Entity와 유사한 Scheme을 가진 class가 framework, bootstrap에 정의되며 값이 매핑되고 있습니다.)

Java에서 이런 객체 변환에 대하여 다양한 매핑 유틸을 찾아볼 수 있지만 [Performance of Java Mapping Frameworks](https://www.baeldung.com/java-performance-mapping-frameworks) Kotlin에서는 신뢰할 만한 util을 찾을 수 없었습니다.

결국 data class 내부에 메서드를 생성하고 모든 필드에 대해 일일이 매핑하는 방법을 선택하게 되었고, 이런 반복적이고 따분한 작업을 도와줄 새로운 멤버를 영입하게 되었습니다.

(고마워요 co-pilot!!)

### Duplicate code

동일한(유사한) Entity나 VO가 각 헥사곤마다 존재하게 되었고 우리는 고민 끝에 각 헥사곤이 자신만의 객체를 보유하게 분리를 결정했지만 잘한 선택이었다고 생각합니다.

각 헥사곤마다 필요로 하는, 그렇지 않은 값들이 존재합니다.

예를 들면 Framework 헥사곤의 DB Entity의 경우 도메인에서는 필요 없는 생성 일시, 생성자, 수정 일시, 수정자들의 값이 포함되며

Bootstrap 헥사곤의 DTO의 경우 도메인에서 사용되는 DateTime 객체를 String format으로 변환해서 내려준다든지 Depth가 깊어 Presentation layer에서 표현이 어려운 경우 적당한 변수명으로 Depth를 낮춰준다든지 자신의 영역에서 전문화된 가공을 할 수 있게 되었습니다.

이는 점차 도메인, 애플리케이션 헥사곤에서 사용하는 객체들이 비즈니스에 집중할 수 있도록 경량화되는 역할이 되기도 하였습니다.

### 세분화된 Config

Bootstrap 헥사곤이 시작될 때 Application 헥사곤에 정의된 usecase들을 bean으로 만들게 되고, 이때 outputPort의 구현체들이 bean으로 정의되어 있지 않는 경우 시작에 실패하게 됩니다. 따라서 Bootstrap 헥사곤에 outputPort의 구현체들이 있는 Framework 헥사곤의 config를 import해야 하는 상황이 생기게 되는데요, 문제는 Application 헥사곤의 모든 usecase를 bean으로 만들었기 때문에 실제 사용하지 않는 Framework 헥사곤의 component들까지도 모두 bean으로 만들지 않으면 안 되는 상황이 생기게 됩니다.

이를 해결하려면 Application 헥사곤에 usecase를 기능별로 구분해서 component scan 경로를 지정해야 하는데 이 방법 또한 하나의 usecase가 다른 usecase에 의존하거나 scan 경로에 없는 outputPort 구현체에 의존하게 될 때 문제가 발생하게 됩니다.

그래서 모든 Application 헥사곤의 usecase를 bean으로 등록하는 것은 유지하되 Bootstrap 헥사곤이 시작될 때 필요한 config만 import될 수 있도록 componentScan option인 lazyInit을 설정하기로 하였습니다.

물론 Runtime에 문제가 발생될 수도 있지만, 테스트 시 충분히 파악할 수 있는 부분이라 이 방법을 유지하기로 했습니다.

### 오늘도 늘어나는 인터페이스

우리는 기존 프로젝트를 새로 만든 프로젝트로 마이그레이션을 진행했습니다. 헥사고날 아키텍처의 특성상 외부 기술과의 연계는 모두 인터페이스를 통해 이루어지기 때문에 패키지를 나눠 기계적으로 코드를 옮겨오는 작업을 하다 보니 수많은 outputPort 인터페이스들이 생겨나게 되었습니다. 그리고 오늘도 수많은 인터페이스가 생겨나고 있습니다.

업무를 진행함에 앞서 usecase를 고민해 보고 어떤 성격의 outputPort를 정의할 것인지 팀 내부에 리뷰가 선행된다면, 중복되거나 불필요한 인터페이스는 점점 사라질 것이라고 생각되어 지금까지와는 약간 다르게 일하는 방법을 새롭게 정립해 나가는 과정을 진행 중에 있습니다.

## 팁

-

도메인 객체의 숙명 유효성 검사

도메인 객체는 기본적으로 그 객체의 유효성 체크가 무엇보다도 중요합니다. 하지만 이때마다 메서드를 만들어 호출하는 것은 bolierplate 코드를 계속 만들어 내는 느낌을 지울 수 없었고 우리는 Kotlin init을 이용하기로 하였습니다.

```
data class CancelOrderDetectionPolicy(
val status: CancelOrderDetectionStatusType,
val notifyThreshold: Int,
val inputMessage: NotificationMessage,
val warningMessage: NotificationMessage,
val temporaryClosureThreshold: Int,
val temporaryClosureMinutes: Long,
val deleted: Boolean
) {
init {
    require(notifyThreshold < temporaryClosureThreshold) { "notifyThreshold must be less than temporaryClosureThreshold" }
    require(notifyThreshold > 0) { "notifyThreshold must be greater than 0" }
    require(temporaryClosureThreshold > 0) { "temporaryClosureThreshold must be greater than 0" }
    require(temporaryClosureMinutes > 0) { "temporaryClosureMinutes must be greater than 0" }
}
}
```

init 영역 안에 기본적인 비즈니스 규칙을 담는다면 헥사곤 간 필드 매핑이 일어날 때 자동으로 규칙 위반 여부를 체크할 수 있었습니다.

-

Audit field는 domain에서 사용되는 값은 아니지만, Admin에 필요

created_at, created_by, modified_at, modified_by 와 같이 DB가 가지는 audit 값은 특정 driven adapter만 가지고 있게 될 필드라고 생각했지만 Bootstrap (Admin)에서 사용될 일이 적지 않게 생기게 되었습니다.

audit field를 Domain/Application 헥사곤 객체들에 포함하기엔 비효율적이라는 생각이 강해 Application 헥사곤에 AuditorApp이라는 generic class를 선언하고 특정 usecase엔 이 class를 사용하기로 하여 불필요한 신규 필드(or class) 생성을 억제하였습니다.

```
data class AuditorApp<T>(
val value: T,
val createdBy: String? = null,
val createdAt: LocalDateTime? = null,
var modifiedBy: String? = null,
var modifiedAt: LocalDateTime? = null
)
```

이제 Admin에서 사용하는 usecase의 메서드는 AuditorApp의 객체를 응답하여 사용하게 되었습니다.(물론 admin bootstrap 헥사곤에선 별도의 DTO를 선언하여 value에 대한 접근성에 편의를 두게 끔 변경하게 되었습니다.)

## 짧은 소회

우리가 구성한 Multi Module이 핵사고날 아키텍처를 100% 구현한 것이라고는 생각하지 않습니다. 이는 패션쇼에서 보이는 옷들이 실제 시장에 나올 때 타깃층을 겨냥하여 콘셉트는 유지하되 적절히 커스터마이징되는 것과 같다고 생각합니다.

이 변화를 시작하기 앞서 내부 구성원들은 더 많은 이야기를 해야 했고, 생각을 공유하며 방향을 정하게 되었습니다. 이러한 과정들이 내부 결속력을 높이며 제품에 대한 오너십을 강하게 만들 수 있었던 계기가 되기도 하였습니다. 그렇기에 우리는 우리가 만들어낸 산출물에 자부심을 가지고 기술 블로그까지 작성하게 된 것은 아닐까 생각합니다.

좋은 애플리케이션 아키텍처를 만드는 것에 대한 결과가 좋은 품질이라고 이야기하긴 어렵습니다. 하물며 헥사고날 아키텍처 도입으로 얻어지는 결과를 정량적으로 평가해야 한다면 그 시도는 더욱 힘들 거라 생각됩니다.

새로운 시도는 좋지만 그 결과에 대한 책임은 녹녹하지 않습니다. 하지만 도전하지 않으면 개인은 물론 조직의 성장도 없다고 생각합니다. 함께 자라기 위해선 도전을 두려워하지 않는 개인, 두려움 없이 도전할 수 있는 환경을 만들어주는 조직 모두 함께 잘해야 한다고 생각합니다.

언제나 그랬듯 알려진 길이 아닌 아무도 가보지 않은 가시밭길을 헤치고 나아가는 사람만이 발전할 수 있다고 생각합니다.

우아한형제들은 이런 것이 가능한 회사라고 생각합니다. 언제든지 더 일을 잘하기 위한 방법을 시도할 수 있는… 고민은 기회를 놓칠 뿐! 우아한형제들의 문을 두드리세요.

-

![](https://techblog.woowahan.com/wp-content/uploads/2025/10/%ED%94%84%EB%A1%9C%ED%95%84-2.png)

[김현겸](https://techblog.woowahan.com/12720/#)

우아한형제들 주문접수채널 서버 개발자
