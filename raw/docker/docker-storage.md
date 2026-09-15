# Storage

> Source: https://docs.docker.com/engine/storage
> Collected: 2026-08-10
> Published: Unknown

Docker storage covers container data persistence and daemon storage backends. Container data persistence stores application data outside containers using volumes, bind mounts, and tmpfs mounts.

By default all files created inside a container are stored on a writable container layer that sits on top of the read-only, immutable image layers. Data written to the container layer doesn't persist when the container is destroyed.

Volumes are persistent storage mechanisms managed by the Docker daemon. They retain data even after the containers using them are removed. Directly accessing or interacting with volume data is unsupported, undefined behavior, and may result in the volume or its data breaking in unexpected ways.

Bind mounts create a direct link between a host system path and a container. Since they aren't isolated by Docker, both non-Docker processes on the host and container processes can modify the mounted files simultaneously.

A tmpfs mount stores files directly in the host machine's memory. This storage is ephemeral and is lost when the container is stopped or restarted, or when the host is rebooted.
