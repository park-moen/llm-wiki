# 프로젝트의 시드 데이터 용어와 병렬 개발 설명

> Source: User-provided project note in conversation
> Collected: 2026-09-17
> Published: 2026-09-17

## 검토 대상 문장

> 시더로 EventTicket·TicketPrice를 먼저 입력하면 어드민 가격 화면(041)과 공개 조회(018)를 병렬로 진행할 수 있다

## 프로젝트 설명

seeder — 앱을 띄울 때 DB에 초기 데이터를 넣어주는 코드입니다. 이 프로젝트는 `@Profile("init")`이 붙은 Kotlin 클래스(`SiteSeeder` 같은)로 관리합니다. 표준 문서는 `be/docs/team-standards/seed-data.md`입니다.

FEAT-041(어드민 가격 화면)이 끝나야 FEAT-018(공개 가격 조회)을 만들 수 있는 게 원래 순서입니다. 어드민에서 가격을 입력해야 조회할 데이터가 생기기 때문입니다.

그런데 `EventTicket`·`TicketPrice` 데이터를 seeder로 미리 코드에 넣으면, 어드민 화면 없이도 조회 화면을 만들 수 있습니다. 즉 041과 018을 동시에 진행해 임계 경로를 줄이자는 뜻입니다.

`시더`는 음역이라 개인 한국어 문체 지침에 어긋납니다. 저장소의 표준 문서는 `시드 데이터`·`seed`를 쓰고 `시더`는 쓰지 않습니다.

다만 `docs/` 안에 `시더`가 여러 파일에 흩어져 있습니다. 한 곳만 `seeder`로 바꾸면 표기가 갈릴 수 있습니다.
