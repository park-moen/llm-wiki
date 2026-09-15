# Port publishing and mapping

> Source: https://docs.docker.com/engine/network/port-publishing/
> Collected: 2026-08-10
> Published: Unknown

By default, for both IPv4 and IPv6, the Docker daemon blocks access to ports that have not been published. Published container ports are mapped to host IP addresses using firewall rules for Network Address Translation, Port Address Translation, and masquerading.

Use the `--publish` or `-p` flag to make a port available outside the host. This creates a firewall rule in the host, mapping a container port to a port on the Docker host.

Publishing container ports is insecure by default. When you publish a container's ports it becomes available not only to the Docker host, but to the outside world as well.

If you include the localhost IP address (`127.0.0.1` or `::1`) with the publish flag, only the Docker host can access the published container port.

By default, when a container's ports are mapped without any specific host address, the Docker daemon publishes ports to all host addresses (`0.0.0.0` and `[::]`), potentially making them available to the outside world.
