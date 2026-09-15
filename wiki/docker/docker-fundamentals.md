# Docker 핵심 개념

> Sources: Docker Documentation, Unknown
> Raw: [What is Docker?](../../raw/docker/docker-overview.md); [Building best practices](../../raw/docker/docker-build-best-practices.md); [Multi-stage builds](../../raw/docker/docker-multi-stage-builds.md); [Storage](../../raw/docker/docker-storage.md); [Networking overview](../../raw/docker/docker-networking-overview.md)
> Updated: 2026-08-10

## Overview

Docker는 애플리케이션을 image로 패키징하고 container로 실행하며 registry를 통해 배포하는 플랫폼이다. Docker client가 API를 통해 Docker daemon에 요청하고, daemon이 image·container·network·volume 같은 객체의 수명주기를 관리한다.

## Docker의 실행 모델

- `docker` client는 사용자의 명령을 Docker API 요청으로 전달한다.
- `dockerd` daemon은 image build, container 실행, network와 volume 관리를 수행한다.
- registry는 image를 저장하고 `pull`·`push`의 대상이 된다.
- Docker Compose도 여러 container로 구성된 애플리케이션을 다루는 Docker client 역할을 한다.

## Image와 container

Image는 container를 생성하기 위한 읽기 전용 template이다. Dockerfile의 instruction은 image layer를 만들며, 재빌드할 때 변경된 layer와 그 영향을 받는 후속 layer가 다시 처리된다.

Container는 image의 실행 가능한 instance다. 같은 image에서 여러 container를 만들 수 있고, 실행 시 network·storage·환경 설정을 별도로 결합한다. Container 내부 writable layer에만 기록한 상태는 container를 제거할 때 사라지므로 지속해야 하는 데이터는 외부 storage에 두어야 한다.

## Image build 기본 원칙

- 신뢰할 수 있고 요구사항에 맞는 작은 base image를 선택한다.
- `.dockerignore`로 build context에 필요하지 않은 파일을 제외한다.
- 불필요한 package를 설치하지 않아 image 크기, dependency와 공격 표면을 줄인다.
- Container는 교체 가능한 일시적 실행 단위로 설계하고 지속 데이터는 외부로 분리한다.
- `--pull`은 더 새로운 base image를 확인하고, `--no-cache`는 기존 build cache를 사용하지 않는다. 두 옵션의 목적은 다르다.

### Multi-stage build

하나의 Dockerfile에서 여러 `FROM`으로 build stage를 나누고 `COPY --from`으로 필요한 artifact만 최종 stage에 옮길 수 있다. 이렇게 하면 compiler, package manager와 중간 산출물을 runtime image에서 제외할 수 있다.

Stage에는 `AS`로 이름을 붙이는 편이 순서 변경에 안전하다. `--target`으로 특정 stage까지만 build할 수 있으므로 development·test·production stage를 하나의 Dockerfile에서 구분할 수도 있다. BuildKit은 선택한 target이 의존하는 stage만 처리한다.

## Storage 선택

| 방식 | 성격 | 적합한 용도 |
|---|---|---|
| Writable container layer | Container별 임시 layer | 제거되어도 되는 실행 중 상태 |
| Volume | Docker daemon이 관리하는 지속 storage | DB 데이터와 장기 보관 데이터 |
| Bind mount | Host path를 container에 직접 연결 | 개발 중 source 공유와 host-container 공동 접근 |
| tmpfs | Host memory에 저장되는 비영속 storage | 임시 cache와 disk에 남기지 않을 데이터 |

Volume의 실제 host 경로를 직접 조작하는 것은 지원되지 않는다. Bind mount는 host와 container가 같은 파일을 동시에 변경할 수 있고 Docker의 격리 밖에 있으므로 쓰기 권한과 대상 경로를 신중하게 제한해야 한다.

## Network 기본 모델

Container는 기본적으로 network 기능과 외부로 나가는 연결을 갖는다. Linux Docker Engine에서 network를 지정하지 않으면 default bridge에 연결된다.

Default bridge에서는 container IP로 통신하지만 이름 기반 탐색이 제공되지 않는다. User-defined network에 연결된 container들은 IP뿐 아니라 container 이름으로도 서로를 찾을 수 있으므로, 서로 통신해야 하는 service 그룹은 전용 network로 명시하는 편이 관리하기 쉽다.

Container port는 publish하기 전에는 일반적으로 host 밖에서 접근할 수 없다. `-p` 또는 `--publish`는 container port를 host에 공개하는 별도 동작이다.

## See Also

- [Docker Compose](docker-compose.md)
- [Docker 보안과 운영 주의점](docker-security-and-operations.md)
