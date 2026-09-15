# Control startup and shutdown order in Compose

> Source: https://docs.docker.com/compose/how-tos/startup-order/
> Collected: 2026-08-10
> Published: Unknown

You can control the order of service startup and shutdown with the `depends_on` attribute. Compose starts and stops containers in dependency order.

On startup, Compose does not wait until a container is "ready", only until it's running. This can cause issues when a service such as a relational database needs to initialize before accepting connections.

Use the `condition` attribute to express one of these dependency states:

- `service_started`
- `service_healthy`
- `service_completed_successfully`

`service_healthy` expects a dependency to be healthy, as defined with `healthcheck`, before starting a dependent service.

Compose removes services in dependency order as well, removing dependent services before their dependencies.
