# Docker Compose

> Sources: Docker Documentation, Unknown
> Raw: [How Compose works](../../raw/docker/docker-compose-application-model.md); [Compose file reference](../../raw/docker/docker-compose-file-reference.md); [Control startup and shutdown order in Compose](../../raw/docker/docker-compose-startup-order.md); [Use Compose in production](../../raw/docker/docker-compose-production.md); [Manage secrets securely in Docker Compose](../../raw/docker/docker-compose-secrets.md)
> Updated: 2026-08-10

## Overview

Docker Compose는 multi-container 애플리케이션을 `compose.yaml`에 선언하고 `docker compose` CLI로 함께 생성·실행·중지하는 도구다. Compose application model은 service, network, volume, config와 secret으로 구성되며 project name으로 한 배포의 resource를 묶고 다른 배포와 구분한다.

## Application model

- `services`: 동일한 container image와 설정을 하나 이상 실행하는 계산 단위
- `networks`: service 사이의 통신 경로
- `volumes`: service가 사용하는 지속 storage
- `configs`: runtime 또는 platform에 의존하는 일반 설정 file
- `secrets`: 민감 정보용 설정 file
- project: 한 Compose application 배포에 속한 resource 묶음

Compose file의 권장 이름은 `compose.yaml`이다. 과거의 `docker-compose.yaml` 이름도 호환되지만 canonical file이 함께 있으면 `compose.yaml`이 우선한다.

현재 권장 형식은 Compose Specification이다. 과거의 2.x와 3.x file format은 하나의 specification으로 통합되었으므로 새 file에서 오래된 형식 구분을 설계 기준으로 삼기보다 현재 Compose Specification과 사용 중인 CLI가 지원하는 field를 확인해야 한다.

## 기본 lifecycle

`docker compose up`은 필요한 container, network와 volume을 만들고 service를 시작한다. `docker compose ps`는 service 상태를 보여주고 `docker compose logs`는 log를 모아 보여준다. `docker compose down`은 해당 project가 생성한 service resource를 중지하고 제거한다.

## 의존성과 readiness

`depends_on`은 service의 생성·제거 순서를 표현하지만 container가 실행 중인 상태와 애플리케이션이 요청을 받을 준비가 된 상태는 다르다. DB처럼 초기화 시간이 필요한 dependency에는 `healthcheck`를 정의하고 `condition: service_healthy`를 사용해야 한다.

일회성 migration처럼 성공적으로 끝나야 하는 선행 작업에는 `service_completed_successfully`를 사용할 수 있다. 단순 process 시작만 필요하다면 `service_started`를 사용한다. 애플리케이션 자체에도 dependency 재연결과 retry 처리가 필요하다.

## Development와 production 구성

동일한 Compose application model을 CI·staging·production에 사용할 수 있지만 development 설정을 그대로 운영 설정으로 간주해서는 안 된다. Production에서는 source code bind mount 제거, host port 재검토, log 설정 조정, restart policy와 log 수집 service 같은 운영 구성이 필요할 수 있다.

공통 `compose.yaml` 위에 `compose.production.yaml`을 나중 순서로 적용해 환경별 변경만 override할 수 있다.

```bash
docker compose -f compose.yaml -f compose.production.yaml up -d
```

여러 Compose file을 병합할 때는 file 순서가 override 결과에 영향을 준다. 최종 적용 모델을 검토하고 version control에 유지하는 것이 중요하다.

## Compose secret

Password, certificate와 API key는 일반 환경변수보다 Compose `secrets`로 service별 접근을 제한하는 편이 안전하다. Top-level `secrets`에서 원본을 정의하고 필요한 service에만 명시적으로 연결한다. Container 안에서는 기본적으로 `/run/secrets/<secret_name>` file로 제공된다.

일부 official image가 지원하는 `*_FILE` 환경변수는 secret file 경로를 image에 전달하기 위한 관례이며 모든 image가 자동 지원하는 기능은 아니다.

## See Also

- [Docker 핵심 개념](docker-fundamentals.md)
- [Docker 보안과 운영 주의점](docker-security-and-operations.md)
