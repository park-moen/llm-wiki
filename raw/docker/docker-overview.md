# What is Docker?

> Source: https://docs.docker.com/get-started/docker-overview/
> Collected: 2026-08-10
> Published: Unknown

Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly.

Docker uses a client-server architecture. The Docker client talks to the Docker daemon, which does the heavy lifting of building, running, and distributing your Docker containers. The Docker client and daemon communicate using a REST API, over UNIX sockets or a network interface.

The Docker daemon (`dockerd`) listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes.

An image is a read-only template with instructions for creating a Docker container. Each instruction in a Dockerfile creates a layer in the image. When you change the Dockerfile and rebuild the image, only those layers which have changed are rebuilt.

A container is a runnable instance of an image. You can connect a container to one or more networks and attach storage to it. When a container is removed, any changes to its state that aren't stored in persistent storage disappear.
