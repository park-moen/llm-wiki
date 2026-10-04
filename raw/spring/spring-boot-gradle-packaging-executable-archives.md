# Packaging Executable Archives

> Source: https://docs.spring.io/spring-boot/gradle-plugin/packaging.html
> Collected: 2026-10-04
> Published: Unknown

## Packaging Executable Archives

The plugin can create executable archives (jar files and war files) that contain all of an application’s dependencies and can then be run with `java -jar`.

## Packaging Executable Jars

Executable jars can be built using the `bootJar` task. The task is automatically created when the `java` plugin is applied and is an instance of `BootJar`. The `assemble` task is automatically configured to depend upon the `bootJar` task so running `assemble` (or `build`) will also run the `bootJar` task.

## Packaging Executable and Plain Archives

By default, when the `bootJar` or `bootWar` tasks are configured, the `jar` or `war` tasks are configured to use `plain` as the convention for their archive classifier. This ensures that `bootJar` and `jar` or `bootWar` and `war` have different output locations, allowing both the executable archive and the plain archive to be built at the same time.

## Packaging Layered Jar or War

By default, the `bootJar` task builds an archive that contains the application’s classes and dependencies in `BOOT-INF/classes` and `BOOT-INF/lib` respectively. Similarly, `bootWar` builds an archive that contains the application’s classes in `WEB-INF/classes` and dependencies in `WEB-INF/lib` and `WEB-INF/lib-provided`.
