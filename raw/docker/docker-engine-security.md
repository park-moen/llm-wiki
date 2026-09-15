# Docker Engine security

> Source: https://docs.docker.com/engine/security/
> Collected: 2026-08-10
> Published: Unknown

There are four major areas to consider when reviewing Docker security: kernel namespaces and cgroups, the Docker daemon attack surface, loopholes in container configuration profiles, and kernel hardening features.

Running containers with Docker implies running the Docker daemon. This daemon requires `root` privileges unless you opt in to Rootless mode.

Only trusted users should be allowed to control your Docker daemon. Docker allows you to share a directory between the Docker host and a guest container without limiting the access rights of the container. A container can mount the host root directory and alter the host filesystem without restriction.

By default, Docker starts containers with a restricted set of capabilities and drops all capabilities except those needed.

One primary risk is that the default set of capabilities and mounts may provide incomplete isolation, either independently or in combination with kernel vulnerabilities.

The best practice is to remove all capabilities except those explicitly required for the container's processes.
