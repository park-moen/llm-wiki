# Manage secrets securely in Docker Compose

> Source: https://docs.docker.com/compose/how-tos/use-secrets/
> Collected: 2026-08-10
> Published: Unknown

A secret is data such as a password, certificate, or API key that shouldn't be transmitted over a network or stored unencrypted in a Dockerfile or source code.

Docker Compose provides a way to use secrets without environment variables. Injecting passwords and API keys as environment variables risks unintentional information exposure. Environment variables can be available to all processes and can be printed in logs while debugging.

Services can only access secrets when explicitly granted by a `secrets` attribute within the `services` top-level element.

Secrets are mounted as a file in `/run/secrets/<secret_name>` inside the container.

First define the secret using the top-level `secrets` element. Then reference it from the service definitions that require access. Compose grants access on a per-service basis.

The `_FILE` environment variables used by some Docker Official Images are an image convention, not a general Compose mechanism.
