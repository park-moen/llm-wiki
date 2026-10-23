# Hexagonal Architecture, 진짜 하실 건가요?

> Source: https://tech.kakaopay.com/post/home-hexagonal-architecture/
> Collected: 2026-10-23
> Published: 2025-01-21

요약: Hexagonal Architecture를 도입했다가 제거한 경험을 통해 해당 아키텍처의 장단점과 실질적인 문제점을 다뤘습니다.
완벽해 보이는 Hexagonal Architecture도 상황에 따라 약점이 될 수 있음을 강조하며, 이를 적절히 활용하는 방법의 중요성을 전달합니다.
이 글은 실제 사례를 기반으로 아키텍처 도입을 고민하는 개발자들에게 유용한 교훈과 인사이트를 제공합니다.

💡 리뷰어 한줄평

bri.ghten Hexagonal Architecture에
대한 너무나 귀중한 경험을 공유한 글로, Hexagonal Architecture 도입을 고려할 때 필요한
부분들을 다시 한번 생각하게 되는 인사이트를 얻을 수 있습니다.

wade.hong 완벽할 거 같았던 Hexagonal
Architecture도 은탄환은 아니었습니다. 그도 약점이 있고 잘 사용하지 않으면 독이 된다는
걸 다시 한번 깨닫게 해 주었습니다. Hexagonal Architecture에서 응용되는 개념을 자신의
상황에 맞게 잘 쓸 수 있다는 점을 배울 수 있습니다.

wonny.p 실제 마주했던 Hexaganal Architecture
문제들을 통해 다시 한번 Hexaganal Architecture에 대해 고찰하게 만드는 실감나는 글이었습니다.

## [시작하며](https://tech.kakaopay.com/post/home-hexagonal-architecture/#시작하며)

안녕하세요, 카카오페이 채널서버유닛 서버 개발자 도리입니다.

채널서버유닛에서는 유저들이 카카오페이의 다양한 서비스에 빠르고 쉽게 접근할 수 있도록 가장 최전방에서 여러 정보를 취합하여 보여주고, 많은 부서와 연동하여 유저에게 편의를 제공하는 서버를 개발하고 있는데요.

이번 글에서는 2023년 개편된 카카오페이의 대문, 카카오페이 새로운 홈 서버의 코드 아키텍처의 변화를 돌아보며, Hexagonal Architecture를 적용했다 다시 제거한 이유에 대해 풀어보려고 합니다.

Hexagonal Architecture가 무엇인지는 알고 계시지만, 도입을 망설이시는 분들께 도움이 되길 바랍니다.

## [2023, 카카오페이 홈 전격 개편](https://tech.kakaopay.com/post/home-hexagonal-architecture/#2023-카카오페이-홈-전격-개편)

카카오페이는 두 가지 앱에서 접근할 수 있는 서비스입니다. 카카오톡에서도 접근할 수 있고, 카카오페이 앱을 통해서도 접근할 수 있죠. 유저들이 각 앱을 통해 접근하면 카카오페이 홈을 만나게 됩니다.

개편 이전에는 카카오톡을 통해 접근했을 때 만나게 되는 UI와 카카오페이 앱을 통해 접근했을 때 만나게 되는 UI가 상이했습니다.
그리고 유저의 데이터 기반으로 보여주는 정보가 많이 없었습니다. 계좌 잔액, 결제 내역을 제외하곤 대부분 정적인 콘텐츠를 보여주는 영역이 많았습니다.

![정적인 콘텐츠가 많았던 홈](https://tech.kakaopay.com/_astro/talk-app-highlight_new.BwSAWI8O_qkQel.webp)
 정적인 콘텐츠가 많았던 홈

원래는 카카오톡에 특화된 기능과 카카오페이앱에 특화된 기능에 조금씩 차이가 있었기 때문에, 채널서버유닛에서는 각 접근 지점별로 마이크로서비스를 구성하여 서버를 관리했습니다.

![카카오톡 / 카카오페이앱 각각 마이크로서비스로 존재하던 홈 서버](https://tech.kakaopay.com/_astro/talk-app-separate.C2UxYC-x_1zc8px.webp)
 카카오톡 / 카카오페이앱 각각 마이크로서비스로 존재하던 홈 서버

2023년 4월부터 점진적으로 배포하기 시작하여 2023년 6월, 카카오톡과 카카오페이 앱 모두 통일된 UI를 만날 수 있도록 전면적으로 카카오페이 홈이 개편되었습니다.

![통합된 홈](https://tech.kakaopay.com/_astro/integrated-home_new.TcYFZaKb_H29QT.webp)
 통합된 홈

통합된 UI를 보여준다는 점과, 기존 두 가지 서버를 동시에 유지보수하는 것이 효율적이지 않다고 생각해 새로운 카카오페이 홈 마이크로서비스를 구성했습니다.

![통합된 마이크로서비스](https://tech.kakaopay.com/_astro/integrated-home-diagram.mzhVmUM1_Z2fSgGu.webp)
 통합된 마이크로서비스

## [카카오페이 홈 서버의 핵심 과제](https://tech.kakaopay.com/post/home-hexagonal-architecture/#카카오페이-홈-서버의-핵심-과제)

개편된 홈의 핵심 과제는 다음과 같았습니다.

#### [Server Driven UI 적용](https://tech.kakaopay.com/post/home-hexagonal-architecture/#server-driven-ui-적용)

개편된 카카오페이 홈은 유저의 앱 버전과 무관하게 최대한 동일한 유저 경험을 제공할 수 있도록 Server Driven UI 기반으로 설계되었습니다. 클라이언트 개발자분들과 일관된 JSON 인터페이스를 정의하고, 해당 인터페이스에 맞춰서 응답을 할 경우 정의된 대로 UI가 보이도록 했습니다.

![Server Driven UI 매핑 예시](https://tech.kakaopay.com/_astro/feed-json-example.BzeL-Hj9_Z1zTglY.webp)
 Server Driven UI 매핑 예시

서버에서 가능한 모든 정보를 제어하도록 설계했고, 콘텐츠들의 순서나 노출 여부 그리고 각 콘텐츠에 노출되어야 하는 텍스트 등이 모두 서버를 통해 제어되고 있습니다.
당시에는 결제탭에서 활용하고 있는 클라이언트실의 SDU 시스템이 없었기 때문에 홈의 특성에 맞춰 직접 스펙을 정의하고 구현했습니다.[1](https://tech.kakaopay.com/post/home-hexagonal-architecture/#user-content-fn-1)

#### [실시간 유저 데이터 서빙](https://tech.kakaopay.com/post/home-hexagonal-architecture/#실시간-유저-데이터-서빙)

개편된 카카오페이 홈은 실시간 데이터를 기반으로 유저에게 이로운 정보를 보여줄 수 있게 설계되었습니다. 또한 홈의 전체 콘텐츠를 Server Driven UI로 구현하기 위해 단일 API로 구성했습니다.
따라서 단일 클라이언트 API 호출에 대해 수많은 연동 API 호출을 통해 정보를 조합해야 하는 서버입니다. 글을 쓰는 현재 연동되어있는 서버는 24개, 평균적으로 유저 1명의 홈을 그려주기 위한 API 호출은 50회에 육박합니다.

![많은 연동을 가진 홈](https://tech.kakaopay.com/_astro/home-client-calls.fBUh1xem_Z1Kr3Jm.webp)
 많은 연동을 가진 홈

## [초기 아키텍처](https://tech.kakaopay.com/post/home-hexagonal-architecture/#초기-아키텍처)

처음 서버를 개발할 때에는 연동을 하기 위해 협의해야 하는 지점도 많았고, 개발해야 하는 양도 많았기 때문에 레이어 단위로 패키징을 한 레이어드 아키텍처 기반으로 개발을 했습니다.

또한 종종 연동 서버에서 받아온 응답을 그대로 클라이언트까지 넘겨줘야 하는 경우도 있었는데, 이때에도 연동 서버의 응답 DTO를 그대로 클라이언트로 보내는 DTO에 재활용하기도 했습니다.

같은 객체가 여러 레이어에 걸쳐 사용되어 불편함을 초래할 때가 종종 있었지만, 익숙한 구조 속에서 빠르게 서비스를 개발할 수 있었습니다.

```
com/kakaopay/home/
│
├── controller/
│   └── CardController.kt
├── service/
│   ├── MoneyCardService.kt
│   └── client/
│       ├── MoneyClient.kt
│       └── SecuritiesClient.kt
└── repository/
```

빠르게 개발을 한 이면에는 여러 문제점도 같이 산재하고 있었는데요.

먼저 프록시를 하는 경우 클라이언트의 UI까지 연동 API 응답에 의존하는 문제가 있었습니다.

앞서 말씀드렸다시피, Server Driven UI 기반으로 설계했기 때문에 클라이언트는 서버에서 응답하는 UI를 그대로 그리는 구조입니다. 연동 API 응답의 스펙이나 값이 바뀔 경우, 값을 검증할 기회 없이 클라이언트로 전달되어 UI가 망가지거나 노출되지 않는 문제가 발생했습니다.

또한 첫 서비스 오픈을 위한 개발이다 보니 연동 API 스펙에 있어 변화도 자주 발생했습니다. API 스펙이 한번 변할 때마다 해당 연동 API 응답을 다시 Server Driven UI 응답으로 변환하기 위해 많은 공수를 들여야 했습니다.

## [외부 의존성 문제, 두 가지 해결책](https://tech.kakaopay.com/post/home-hexagonal-architecture/#외부-의존성-문제-두-가지-해결책)

이런 외부 의존성 문제를 해결하기 위해 두 가지 해결책을 고민했습니다. 하나는 다른 팀과의 협업 ‘프로세스’를 통한 해결이었고, 다른 하나는 우리 코드의 ‘아키텍처’를 통한 해결이었습니다.

### [해결책 1: 프로세스를 통한 해결, ‘연동 API 인터페이스’](https://tech.kakaopay.com/post/home-hexagonal-architecture/#해결책-1-프로세스를-통한-해결-연동-api-인터페이스)

첫 번째 접근은 ‘연동 API 인터페이스’ 라는 이름의 표준 API 스펙을 정의하는 것이었습니다.

보통 서버 간 연동이 필요한 경우는 다른 부서 실시간 정보를 불러와 아래와 같은 피드 형태로 변환하는 경우입니다.

![피드 구조](https://tech.kakaopay.com/_astro/feed-structure_new.4YokUiI9_Z1kUsCY.webp)
 피드 구조

개선 이전에는 기존에 존재하던 API의 응답 값을 홈 서버의 로직으로 처리하여 해당 카드 형태로 변환하고 있었습니다. 가령, 페이포인트 카드를 예로 들어 보면 아래와 같았습니다.

![페이포인트 변환 예시](https://tech.kakaopay.com/_astro/feed-convert.CFteKCqr_1Jn456.webp)
 페이포인트 변환 예시

이 경우 연동 API 스펙이 변경되면 필연적으로 홈 서버에서도 반영을 위한 작업을 해야 합니다. 특히 전사적으로 큰 기술 변경이 있을 때마다 홈 서버는 연동하는 모든 부서의 응답 변경에 대응해야 하는 경우도 발생할 수 있습니다.

이를 해결하기 위해 연동 부서에서 특정 스펙에 맞춰 홈 서버에 API에 응답하면 약속된 형태로 클라이언트에 반환할 수 있는 API 스펙을 만들었습니다. 홈 서버에서는 어드민에 등록되어 있는 정보와 해당 연동 API 응답을 교차 검증, 활용하여 카드 응답을 생성했습니다.

일종의 데이터 생성에 대한 책임을 다른 부서에 위임한 것과도 같다고 볼 수 있었습니다. 콘텐츠를 어떻게 보여줄지에 대한 책임이 각 연동 부서로 넘어가면서, 홈 서버는 외부 서비스의 세세한 변경에 일일이 대응할 필요가 없어졌습니다. 다른 부서에서 보여주고 싶은 콘텐츠가 바뀌어도, 인터페이스만 잘 지켜진다면 홈 서버의 별도 작업 없이 반영하는 형태를 갖출 수 있었습니다.

![연동 API 인터페이스 적용 예시](https://tech.kakaopay.com/_astro/feed-api-interface.DR-2K_3o_13eFcy.webp)
 연동 API 인터페이스 적용 예시

### [해결책 2: 아키텍처를 통한 해결, ‘Hexagonal Architecture’](https://tech.kakaopay.com/post/home-hexagonal-architecture/#해결책-2-아키텍처를-통한-해결-hexagonal-architecture)

두 번째 접근은 우리 코드 내부 아키텍처를 통해 해결하는 것이었습니다. Hexagonal Architecture가 이것에 적합하다고 생각했습니다.

연동 API 인터페이스가 모든 문제를 해결해주진 못할 거라 생각했습니다. 여전히 인터페이스에 맞지 않는 레거시 API들도 존재했고, 앞으로 어떤 새로운 연동이 추가될지 몰랐기 때문입니다. 그래서 외부의 변화가 우리 서버의 핵심 로직(도메인)에 침투하지 못하도록 막는 구조적 안전장치가 필요하다고 판단했습니다.

이를 위해 많은 연동 API들을 각기 다른 out Port로 추상화하여 Hexagonal Architecture를 적용했고, 멀티 모듈을 적용하여 의존성 방향을 엄격하게 강제했습니다.

```
com/kakaopay/home/
│
├── api/
│   ├── adapter/
│   └── controller/
├── domain/
│   ├── port/
│   └── service/
└── infrastructure/
    ├── adapter/
    ├── repository/
    └── client/
```

각 모듈별 역할은 다음과 같이 배치했습니다.

- api: domain의 in Port를 통해 반환받은 도메인 객체를 API 응답에 맞는 형태로 변환, Spring Boot 서버 구동 역시 담당

- domain: Port와 해당 Port들을 활용하여 도메인 객체를 생성

- infrastructure: 외부 API 연동 담당, 연동 API 응답을 out Port에서 지원하는 형태로 변환

![카카오페이 홈에 적용한 Hexagonal Architecture](https://tech.kakaopay.com/_astro/hexagonal-architecture.BcoO1Cfu_Z1QhHh6.webp)
 카카오페이 홈에 적용한 Hexagonal Architecture

Hexagonal Architecture 적용 이후에는 연동 부서에서 API 응답을 변경하거나 혹은 연동해야 하는 API 자체가 달라지더라도 infrastructure 모듈만 수정하여 in Port 인터페이스에만 맞추면 기존 도메인 로직은 변경할 필요 없이 로직을 지켜낼 수 있었습니다. 도메인 로직의 재사용성을 높이면서도, 외부 세계의 변동에 대해 빠르게 대응할 수 있을 것이라는 기대감에 부풀었습니다.

## [이상과 현실의 충돌: 운영 과정에서 드러난 문제점](https://tech.kakaopay.com/post/home-hexagonal-architecture/#이상과-현실의-충돌-운영-과정에서-드러난-문제점)

하지만 가장 이상적일 거라 생각했던 우리의 구조는 현실에서 충돌하기 시작했습니다. 기대했던 장점보다는 운영의 비효율성이 점점 더 커졌습니다.

### [두 해결책의 모순](https://tech.kakaopay.com/post/home-hexagonal-architecture/#두-해결책의-모순)

가장 근본적인 문제는, 두 개의 해결책이 서로의 존재 이유를 약화시킨다는 점이었습니다.

애초에 연동 부서의 변경으로부터 독립되는 문제를 ‘연동 API 인터페이스’라는 프로세스로 해결하고 있었기 때문에, Hexagonal Architecture를 적용해도 코드 구조를 통해 얻는 효용이 크게 떨어졌습니다.

이미 연동 인터페이스가 외부 변화를 막아주는 훌륭한 방파제 역할을 하고 있었기 때문에 헥사고날 아키텍처의 핵심인 ‘도메인 로직 보호’라는 장점이 퇴색될 수 밖에 없었습니다. 보호해야 할 대상이 이미 다른 방식으로 보호받고 있었던 것입니다.

### [애매한 도메인에 대한 정의](https://tech.kakaopay.com/post/home-hexagonal-architecture/#애매한-도메인에-대한-정의)

이러한 모순은 ‘도메인’의 정의를 애매하게 만들었습니다. 홈 서버의 핵심 비즈니스 로직은 사실상 ‘외부 데이터를 조합하여 SDUI에 맞게 렌더링하는 것’이었습니다. 즉, 순수한 의미의 독립적인 도메인 모델이라기보다는 외부 데이터를 ‘활용’하고 ‘보여주는’ 로직에 가까웠습니다.

결국 보호해야 할 핵심 도메인이 명확하지 않으니, Hexagonal Architecture의 존재 이유도 점차 희미해졌습니다. 저희가 핵심 로직이라고 생각했던 부분들이 사실은 도메인 로직과는 거리가 멀고, 오히려 외부 세계와 소통하는 Adapter의 로직에 가까웠던 것이죠.

### [모호한 레이어 간 경계](https://tech.kakaopay.com/post/home-hexagonal-architecture/#모호한-레이어-간-경계)

핵심 도메인이 애매하다 보니, 자연스럽게 Port와 Adapter의 경계도 모호해지기 시작했습니다. out Port의 인터페이스는 무엇을 기준으로 만들어야 할까요? 결국 ‘지켜내야 할 도메인’이 없으니, 기존에 연동하던 WebClient의 응답을 기준으로 Port를 작성하게 되었습니다.

이는 외부 세계(infrastructure)의 구현이 내부의 인터페이스(domain)를 정의하는 주객전도 현상을 낳았습니다. 외부의 변화가 Port에까지 영향을 미치게 되면서, 변경으로부터 도메인을 지킨다는 아키텍처의 기본 전제가 무너지는 상황이 자주 발생했습니다.

### [신규 기능 개발의 비효율성](https://tech.kakaopay.com/post/home-hexagonal-architecture/#신규-기능-개발의-비효율성)

신규 기능 개발과 기존 API 변경 사이의 비중 변경도 효용에 영향을 미쳤습니다.

연동 인터페이스의 적극적인 도입에 힘입어 연동 API의 변경이 홈 서버의 작업으로 이어지는 빈도는 급격히 줄어갔습니다. 반면 홈 서버의 신규 기능 개발 빈도는 점점 증가했습니다.

신규 기능의 개발은 곧 전체 레이어에 대한 변경을 의미했습니다. 다음의 변경이 대부분 필수적으로 발생했습니다.

- 도메인 레이어에 로직 추가/변경

- 도메인 레이어에서 사용할 수 있는, 사용할 out/in Port 인터페이스 추가/변경

- 해당 Port들의 인터페이스에 맞춘 Adapter 개발

이 Port에 값을 제공해 줄 클라이언트로부터 값을 받아 처리하려면 다음의 과정을 거쳐야 했습니다.

![변환 구조도](https://tech.kakaopay.com/_astro/convert-diagram.Bj2PZxZR_2m93XD.webp)
 변환 구조도

신규 기능을 개발할 때에는 이런 구조를 가진 여러 파일에 Domain -> Port -> Adapter 순으로 작업을 해야 했습니다. 또한 각각의 레이어가 가진 DTO, 그리고 매핑 로직까지 모두 작업을 해야 했죠.

카카오페이 홈 서버는 개편을 진행한 2023년 기준, 연동 서버 23개 대상으로 40개 이상의 API를 호출하고 있었습니다. 그리고 각각의 케이스마다 다른 기능이 사용되었기 때문에 40개 이상의 out Port가 존재했는데요.
신규 기능을 빠르게 개발해야 하는 상황에서 도메인 레이어부터 시작해서 다수의 out Port, DTO, 그리고 매핑 로직을 수정해야 했습니다. 이러한 작업은 시간이 지나 Port가 많아질수록 점점 더 큰 비용이 되어갔습니다.

### [단일 in Port](https://tech.kakaopay.com/post/home-hexagonal-architecture/#단일-in-port)

카카오페이 홈 서버는 현재로선 HTTP API를 제공하는 컴포넌트 하나만 구동이 되고 있습니다. 다시 말해 도메인이 있다고 하더라도 해당 모듈을 재사용하는 모듈이 없다는 것이죠. 물론 처음 Hexagonal Architecture를 적용할 때에는 같은 도메인 로직을 활용하는 유저의 데이터를 미리 적재하는 Consumer 컴포넌트가 추가될 수도 있겠다는 생각을 했습니다. 하지만 운영하는 시간이 길어질수록 만약 Consumer 컴포넌트를 만든다고 하더라도 Consumer에서 활용해야 하는 로직과 API 서버에서 활용해야 하는 로직이 각자의 책임에 따라 크게 차이가 날 것이라는 생각이 들었습니다.

따라서 도메인 모듈의 재사용성이 사실상 크게 떨어지는 것이고, 모듈을 재사용하지 못한다면 Hexagonal Architecture를 적용하는 의미도 크게 떨어진다고 판단했습니다.

### [팀 온보딩 및 코드 관리 비용 증가](https://tech.kakaopay.com/post/home-hexagonal-architecture/#팀-온보딩-및-코드-관리-비용-증가)

팀에 새로운 동료가 생길 때마다 홈 서버의 아키텍처를 설명하는 부분도 점점 어려워졌습니다. Hexagonal Architecture라는 설명으로는 부족할 정도로 하는 기능에 비해 코드의 사이즈가 비대해진 나머지 하나의 기능이 어떻게 동작하는지 설명하기 위해 많은 비용을 들여야 했습니다.

또한 Hexagonal Architecture에 대해 팀원들이 이해한 방향이 조금씩 다른 경우가 많아서 구조적인 강제성을 띄어야 할 아키텍처가 오히려 더 많은 다양성을 만들어냈습니다. 결과적으로 안 그래도 많아진 파일에 산발적으로 로직이 흩어지며 관리 비용이 증가했습니다.

## [내부의 정비가 먼저: Hexagonal Architecture 제거](https://tech.kakaopay.com/post/home-hexagonal-architecture/#내부의-정비가-먼저-hexagonal-architecture-제거)

저희는 Hexagonal Architecture가 제공하는 구조적 장점보다, 운영과 개발 과정에서의 비효율성이 더 크다고 결론 내렸습니다.

또한 유저에게 더 나은 가치를 제공하기 위해서라면 어떤 형태로 변화될지 예상할 수 없는 프로젝트 특성을 고려하여 과감히 Hexagonal Architecture 제거를 결정했습니다.

기존 ‘연동 인터페이스’에 맞춰 연동해둔 수많은 API를 다시 수정 개발 요청하는 방법도 분명 있었습니다. 하지만 이를 위해 수정 개발을 요청해야 하는 부서가 너무 많았기 때문에 아키텍처를 다시 변경하는 내부의 정비를 먼저 수행하기로 했습니다.

### [제거 과정](https://tech.kakaopay.com/post/home-hexagonal-architecture/#제거-과정)

제거 과정에서는 다음의 부분들을 고려했습니다.

#### [관심사 단위 패키징](https://tech.kakaopay.com/post/home-hexagonal-architecture/#관심사-단위-패키징)

최초 개발 시 적용했던 레이어 단위 패키징으로 돌아가는 것이 아니라, 관심사 단위 패키징을 적용하기로 했습니다. Hexagonal Architecture에서 도메인 모델로 인식했던 것들을 기준으로 패키징을 적용하여 로직을 담당하는 코드를 더 빨리 탐색할 수 있게 했습니다.

#### [단일 모듈 구조로 회귀](https://tech.kakaopay.com/post/home-hexagonal-architecture/#단일-모듈-구조로-회귀)

관심사 단위 패키징을 적용하면서 Controller, Service, Infrastructure 관련 의존성을 하나의 패키지 하위에 위치하는 것이 자연스럽겠다는 생각을 했습니다. 따라서 멀티 모듈을 단일 모듈로 합치는 작업을 가장 먼저 수행했습니다. 기존 Hexagonal Architecture를 적용하면서도 Component Scan을 지원하기 위해 Annotation만 gradle 의존성으로 가지고 있었던 부분들이 자연히 해결되었고, 의존성 간 버전이 어긋나는 것 역시 자연히 해결되었습니다.

#### [Port와 DTO 처리 방식 재설계](https://tech.kakaopay.com/post/home-hexagonal-architecture/#port와-dto-처리-방식-재설계)

Port를 통해 받아오던 DTO 로직을 통합하고 매핑 로직을 제거하는 작업은 굉장히 크기가 큰 작업이었습니다.
의존성의 관점으로 볼 때, 기존 로직에서 사용하는 필드들은 모두 Port의 인터페이스에 정의되어 있는 DTO에 있을 것입니다. 그리고 Port에 맞춰 매핑하기 위한 WebClient의 DTO는 JSON 이름을 기준으로 작성되어 있을 것입니다.
이 DTO들과 Adapter에서의 변환을 통합하기 위해 Port의 DTO를 기준으로 가장 필요한 필드들만 남기고, WebClient의 DTO를 기준으로 필드의 이름을 설정했습니다. 그리고 기존에 있던 변환 로직의 경우 Port를 사용하던 Service가 처리하도록 했습니다.

통상적으로 많이 사용하는 접근이지만, Port와 Adapter, 그리고 Adapter가 사용하는 WebClient로 책임이 분리되어 있던 것을 다시 통합하는 과정이다 보니 이러한 기준이 필요했습니다.

#### [Controller와 Infrastructure 간 DTO 분리 유지](https://tech.kakaopay.com/post/home-hexagonal-architecture/#controller와-infrastructure-간-dto-분리-유지)

Hexagonal Architecture를 적용했을 때 느꼈던 엄격한 레이어 간 DTO 분리는 시스템의 안정성을 높이는 데에 크게 기여했음이 틀림없었습니다.

따라서 클라이언트에 연동 API의 응답을 그대로 응답하더라도 Controller와 Infrastructure에서 사용하는 DTO를 분리하는 것은 엄격히 유지했습니다. 또한 데이터베이스 접근과 연동 API에서 활용되는 Image, Link 등의 응답에 대해서도 공통으로 활용하지 않고 모두 분리했습니다. 중복 코드로 볼 수도 있지만, 조직의 관점에서 각각 서로 다른 조직과 협업하는 것이기 때문에 벽을 치듯 최대한 분리했습니다.
이를 위해 응답에 대한 DTO 클래스를 Namespace로 인식하고 Nested Class를 활용하여 혼동을 최소화했습니다.

### [제거 후 변화](https://tech.kakaopay.com/post/home-hexagonal-architecture/#제거-후-변화)

Hexagonal Architecture를 제거한 이후 현재까지의 패키지 구조는 아래와 같습니다.

```
com/kakaopay/home/
│
├── cards/
│   ├── controller/
│   └── service/
└── card/
    ├── money/
    │   ├── service/
    │   └── infrastructure/
    ├── shortcut/
    │   ├── service/
    │   └── infrastructure/
    └── feed/
        ├── point/
        │   ├── service/
        │   └── infrastructure/
        ├── money/
        │   ├── service/
        │   └── infrastructure/
        └── mydata/
            ├── service/
            └── infrastructure/
```

관심사별로 구분되어 있는 레이어드 아키텍처의 패키지 구조를 따라가고 있습니다. 큰 변화가 없는 것으로 느껴지실 수도 있지만, PR 기준 8000줄 이상의 코드가 줄어들었습니다. 줄어든 코드의 양만큼 로직을 파악하기 편해졌고, 파악하기 편해진 만큼 신규 기능을 추가하는 데에 걸리는 시간도 줄어들었습니다.

Hexagonal Architecture 제거 이후 곧바로 Controller부터 Infrastructure까지 전체 레이어에 추가 개발을 해야 하는 과제를 진행했는데요. 카카오페이 증권과 협업이 필요한 과제여서 컴플라이언스 이슈, 네트워크 프록시 이슈 등 다양한 이슈 아래에서 개발 기간이 굉장히 촉박했음에도 불구하고 필요한 부분만 효율적으로 변경하여 무리 없이 기능을 릴리즈할 수 있었습니다.

## [Hexagonal Architecture, 어떤 프로젝트에 적합할까?](https://tech.kakaopay.com/post/home-hexagonal-architecture/#hexagonal-architecture-어떤-프로젝트에-적합할까)

카카오페이 홈보다 Hexagonal Architecture가 적합하다고 생각하는 프로젝트의 특징은 다음과 같습니다.

### [도메인 모델을 확실하게 정의할 수 있는 서비스](https://tech.kakaopay.com/post/home-hexagonal-architecture/#도메인-모델을-확실하게-정의할-수-있는-서비스)

도메인 모델이 명확하지 않은 경우, Hexagonal Architecture의 핵심 요소인 Port와 Adapter의 경계를 설정하는 데에서 어려움이 발생할 수 있습니다.

따라서 하나의 프로젝트 내에서 관리해야 하는 도메인 모델을 명확히 정의하는 것이 Hexagonal Architecture 적용의 시작이 되어야 한다고 생각합니다. 더불어 Hexagonal Architecture에 있어 DDD가 필수는 아니지만, 그에 준하게 필요하다는 생각 역시 들었습니다.

### [외부 의존성이 많지 않은 서비스](https://tech.kakaopay.com/post/home-hexagonal-architecture/#외부-의존성이-많지-않은-서비스)

의존성이 많지 않다는 것은 깊이에 대한 부분이 아니라 넓이에 대한 부분입니다. 로직의 대부분이 연동 API에 의해 동작하는 서비스일 경우 결론적으로 Port와 Adapter에 로직이 과중되는 경우가 많아질 확률이 높다고 생각합니다. 따라서 외부에 대한 의존성보다 코어 로직이 풍부하게 존재하는 서비스일 때 의미를 가지게 된다고 생각합니다.

이는 위에서 말한 도메인 모델을 확실하게 정의할 수 있단 의미와 거의 비슷한 것 같기도 합니다.

### [코어 모듈을 사용하는 모듈이 2개 이상인 서비스](https://tech.kakaopay.com/post/home-hexagonal-architecture/#코어-모듈을-사용하는-모듈이-2개-이상인-서비스)

코어 모듈을 2개 이상의 서비스가 사용하지 않으면 out Port에 해당하는 부분만 추상화하는 것이 더 나은 방향이 될 수 있다고 생각합니다. 코어 모듈의 로직을 재사용할 수 있도록 해당 모듈을 사용하는 컴포넌트 모듈이 2개 이상인 서비스 일 때 Hexagonal Architecture를 고려해봐야 한다고 생각합니다.

## [마치며](https://tech.kakaopay.com/post/home-hexagonal-architecture/#마치며)

“은탄환은 없다”는 말, 소프트웨어를 개발하면 어느 상황에서든 적용되는 말이라고 생각하는데요.

Hexagonal Architecture는 좋은 총과 저격수 그리고 알맞은 환경이 주어지면 은탄환에 가까울 수 있겠지만, 적어도 카카오페이 홈에 있어서는 은탄환이 아니었습니다.

Hexagonal Architecture 적용을 고민하고 계신다면, 구조적인 이점만을 보기보다는 프로젝트의 본질적인 특성과 도메인에 대한 이해가 필수적으로 선행되어야 한다고 생각합니다.

제 사례가 그 길을 고민하는 분들께 한번쯤 재고하는 데에 도움이 되었으면 좋겠습니다.

감사합니다.

## [각주](https://tech.kakaopay.com/post/home-hexagonal-architecture/#각주)

## [Footnotes](https://tech.kakaopay.com/post/home-hexagonal-architecture/#footnote-label)

-

결제탭의 SDU 시스템과 아키텍처에 대해선 [이 글](https://tech.kakaopay.com/post/payment-feed-server/)을 참조해주세요! [↩](https://tech.kakaopay.com/post/home-hexagonal-architecture/#user-content-fnref-1)
