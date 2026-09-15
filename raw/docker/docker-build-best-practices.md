# Building best practices

> Source: https://docs.docker.com/build/building/best-practices/
> Collected: 2026-08-10
> Published: Unknown

The first step towards achieving a secure image is to choose the right base image. When choosing an image, ensure it's built from a trusted source and keep it small.

When building your own image from a Dockerfile, ensure you choose a minimal base image that matches your requirements. A smaller base image not only offers portability and fast downloads, but also shrinks the size of your image and minimizes the number of vulnerabilities introduced through the dependencies.

The `--pull` flag forces Docker to check for and download a newer version of the base image, even if you have a version cached locally.

The `--no-cache` flag disables the build cache, forcing Docker to rebuild all layers from scratch. It does not pull a fresh base image — for that, use `--pull`.

To exclude files not relevant to the build, without restructuring your source repository, use a `.dockerignore` file.

The image defined by your Dockerfile should generate containers that are as ephemeral as possible.

Avoid installing extra or unnecessary packages. When you avoid unnecessary packages, your images have reduced complexity, dependencies, file sizes, and build times.
