# How Compose works

> Source: https://docs.docker.com/compose/intro/compose-application-model/
> Collected: 2026-08-10
> Published: Unknown

With Docker Compose you use a YAML configuration file, known as the Compose file, to configure your application's services, and then create and start all the services with the Compose CLI.

The Compose file, or `compose.yaml` file, follows the rules provided by the Compose Specification in how to define multi-container applications.

Computing components of an application are defined as services. Services communicate with each other through networks and store or share persistent data in volumes. Configs provide runtime or platform-dependent configuration data. Secrets are a specific flavor of configuration data for sensitive data.

A project is an individual deployment of an application specification on a platform. A project's name is used to group resources together and isolate them from other applications or other installations of the same Compose application.

The preferred default path is `compose.yaml` or `compose.yml`. The older `docker-compose.yaml` and `docker-compose.yml` names are supported for backwards compatibility.

The `docker compose up` command starts services and creates the necessary networks and volumes. `docker compose down` stops and removes them.
