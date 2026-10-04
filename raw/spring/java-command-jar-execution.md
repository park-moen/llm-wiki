# The java Command — Running a JAR

> Source: https://docs.oracle.com/en/java/javase/26/docs/specs/man/java.html
> Collected: 2026-10-04
> Published: Unknown

To launch the main class in a JAR file:

`java` [options] `-jar` jarfile [args ...]

`-jar` jarfile

Executes a program encapsulated in a JAR file. The jarfile argument is the name of a JAR file with a manifest that contains a line in the form `Main-Class:`classname that defines the class with the `public static void main(String[] args)` method that serves as your application's starting point.
