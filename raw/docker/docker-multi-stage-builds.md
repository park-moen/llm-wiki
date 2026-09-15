# Multi-stage builds

> Source: https://docs.docker.com/build/building/multi-stage/
> Collected: 2026-08-10
> Published: Unknown

Multi-stage builds are useful for optimizing Dockerfiles while keeping them readable and maintainable.

With multi-stage builds, you use multiple `FROM` statements in your Dockerfile. Each `FROM` instruction can use a different base, and each begins a new stage of the build.

You can selectively copy artifacts from one stage to another, leaving behind everything you don't want in the final image. Build tools and intermediate artifacts can be left out of the resulting production image.

Build stages can be named by adding `AS <NAME>` to the `FROM` instruction and referenced by name in `COPY --from=<NAME>`. Named references remain stable if Dockerfile instructions are reordered.

You can stop at a specific stage with `docker build --target <NAME>`. This can support separate debugging, testing, and lean production stages.

BuildKit only builds the stages that the selected target depends on, while the legacy builder processes every stage leading up to the target.
