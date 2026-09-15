# Use Compose in production

> Source: https://docs.docker.com/compose/how-tos/production/
> Collected: 2026-08-10
> Published: Unknown

When you define your app with Compose in development, you can use this definition in CI, staging, and production.

The easiest way to deploy an application is to run it on a single server, similar to a development environment.

Production changes may include removing volume bindings for application code, binding to different host ports, setting environment variables differently, specifying a restart policy, and adding services such as a log aggregator.

Consider defining an additional Compose file such as `compose.production.yaml` containing production-specific changes. The additional file is applied over the original `compose.yaml`:

```bash
docker compose -f compose.yaml -f compose.production.yaml up -d
```

When application code changes, rebuild the image and recreate the application's containers.

A remote Docker host can be addressed with the `DOCKER_HOST`, `DOCKER_TLS_VERIFY`, and `DOCKER_CERT_PATH` environment variables.
