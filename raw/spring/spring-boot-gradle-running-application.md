# Running your Application with Gradle

> Source: https://docs.spring.io/spring-boot/gradle-plugin/running.html
> Collected: 2026-10-04
> Published: Unknown

To run your application without first building an archive use the `bootRun` task:

`$ ./gradlew bootRun`

The `bootRun` task is an instance of `BootRun` which is a `JavaExec` subclass. As such, all of the usual configuration options for executing a Java process in Gradle are available to you. The task is automatically configured to use the runtime classpath of the main source set.
