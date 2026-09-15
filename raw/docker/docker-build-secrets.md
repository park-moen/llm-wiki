# Build secrets

> Source: https://docs.docker.com/build/building/secrets/
> Collected: 2026-08-10
> Published: Unknown

A build secret is any piece of sensitive information, such as a password or API token, consumed as part of your application's build process.

Build arguments and environment variables are inappropriate for passing secrets to your build, because they persist in the final image. Instead, you should use secret mounts or SSH mounts, which expose secrets to your builds securely.

Secret mounts are general-purpose mounts for passing secrets into your build. A secret mount takes a secret from the build client and makes it temporarily available inside the build container, for the duration of the build instruction.

For secret mounts and SSH mounts, using build secrets is a two-step process. First you need to pass the secret into the `docker build` command, and then you need to consume the secret in your Dockerfile.

To pass a secret to a build, use the `docker build --secret` flag. To consume a secret in a Dockerfile, use the `--mount=type=secret` flag in a `RUN` instruction.

The default file path of a secret inside the build container is `/run/secrets/<id>`.
