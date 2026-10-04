# The Java Plugin — Tasks

> Source: https://docs.gradle.org/current/userguide/java_plugin.html
> Collected: 2026-10-04
> Published: Unknown

## Tasks

`jar` — Jar

Depends on: `classes`

Assembles the production JAR file, based on the classes and resources attached to the `main` source set.

## Lifecycle Tasks

`assemble`

Depends on: `jar`

Aggregate task that assembles all the archives in the project. This task is added by the Base Plugin.

`build`

Depends on: `check`, `assemble`

Aggregate tasks that performs a full build of the project. This task is added by the Base Plugin.
