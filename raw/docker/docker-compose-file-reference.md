# Compose file reference

> Source: https://docs.docker.com/reference/compose-file/
> Collected: 2026-08-10
> Published: Unknown

The Compose Specification is the latest and recommended version of the Compose file format. It helps you define a Compose file used to configure your Docker application's services, networks, volumes, and more.

Legacy versions 2.x and 3.x of the Compose file format were merged into the Compose Specification. It is implemented in versions 1.27.0 and above, also known as Compose v2, of the Docker Compose CLI.

The major top-level elements described by the specification include:

- `name` and `version`
- `services`
- `networks`
- `volumes`
- `configs`
- `secrets`
