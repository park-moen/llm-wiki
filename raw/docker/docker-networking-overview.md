# Networking overview

> Source: https://docs.docker.com/engine/network/
> Collected: 2026-08-10
> Published: Unknown

Container networking refers to the ability for containers to connect to and communicate with each other, and with non-Docker network services.

Containers have networking enabled by default, and they can make outgoing connections. A container only sees a network interface with an IP address, a gateway, a routing table, DNS services, and other networking details.

When Docker Engine on Linux starts for the first time, it has a single built-in network called the "default bridge" network. When you run a container without the `--network` option, it is connected to the default bridge.

With the default configuration, containers attached to the default bridge network have unrestricted network access to each other using container IP addresses. They cannot refer to each other by name.

You can create custom, user-defined networks and connect groups of containers to the same network. Once connected to a user-defined network, containers can communicate with each other using container IP addresses or container names.

Use the `--publish` or `-p` flag to make a port available outside the host and to containers in other bridge networks.
