# Docker 보안과 운영 주의점

> Sources: Docker Documentation, Unknown
> Raw: [Docker Engine security](../../raw/docker/docker-engine-security.md); [Port publishing and mapping](../../raw/docker/docker-port-publishing.md); [Build secrets](../../raw/docker/docker-build-secrets.md); [Manage secrets securely in Docker Compose](../../raw/docker/docker-compose-secrets.md)
> Updated: 2026-08-10

## Overview

Container는 격리 계층이지만 독립된 보안 경계라고 단정할 수는 없다. Docker daemon 권한, host mount, Linux capability와 kernel 취약점이 결합될 수 있으므로 daemon 접근을 제한하고 각 container에 최소 권한만 부여해야 한다.

## Docker daemon은 강한 권한 경계다

일반적인 Docker daemon은 Rootless mode를 선택하지 않는 한 root 권한으로 동작한다. Docker를 제어할 수 있는 사용자는 host filesystem을 container에 mount하고 변경할 수 있으므로 Docker daemon과 socket 접근 권한은 사실상 host 관리자 권한으로 취급해야 한다.

## Container 권한 최소화

Docker는 기본적으로 제한된 Linux capability 집합으로 container를 시작하지만, 기본 capability와 mount 조합도 불완전한 격리가 될 수 있다. 필요한 capability만 남기고 불필요한 capability를 제거하며, 광범위한 host mount와 과도한 권한 부여를 피해야 한다.

## Port publish는 외부 노출이다

Host 주소를 생략한 `-p 8080:80` 같은 mapping은 기본적으로 모든 host interface에 port를 publish할 수 있다. Local 개발 도구처럼 host에서만 접근해야 한다면 `127.0.0.1:8080:80`처럼 loopback 주소를 명시하고, 운영 환경에서는 host firewall과 network policy까지 함께 검토해야 한다.

`EXPOSE`로 image의 의도된 port를 문서화하는 것과 `-p`로 실제 host port를 공개하는 것은 구분해야 한다.

## Build secret을 image에 남기지 않기

Build 과정에서 필요한 password, API token과 SSH credential을 `ARG`나 `ENV`로 전달하면 최종 image에 남을 수 있다. BuildKit secret mount 또는 SSH mount를 사용해 해당 `RUN` instruction 동안만 노출해야 한다.

Secret은 `docker build --secret`으로 builder에 전달하고 Dockerfile에서는 `RUN --mount=type=secret`으로 소비한다. 기본 file mount 위치는 `/run/secrets/<id>`다.

Runtime secret도 일반 환경변수에 직접 넣으면 process 환경이나 debug log를 통해 노출될 수 있다. Compose에서는 필요한 service에만 `secrets` 접근을 부여하고 file mount로 제공한다. Source secret file 자체의 보관·권한·배포 경로는 Compose 밖에서도 별도로 보호해야 한다.

## See Also

- [Docker 핵심 개념](docker-fundamentals.md)
- [Docker Compose](docker-compose.md)
