---
publish: true
title: Build docker image  and run docker container on MacOS M1
aliases: []
date: 2024-10-21T14:31:33Z
lastmod: 2024-10-21T20:20:23Z
tags: []
category: posts
summary: 
---

## Build docker image for macOS M1

```
docker buildx build --platform linux/arm64 --load -t rcore_lab .
```

### Explanation

- `docker buildx build`: This initiates the Docker Buildx build process. Docker Buildx is a CLI plugin that extends the functionality of Docker's build command, allowing for advanced features like multi-platform builds, cache import/export, and more.

- `--platform linux/arm64`: This flag specifies the target platform for the build. In this case, it is set to `linux/arm64`, which means the Docker image will be built for the ARM64 architecture on a Linux operating system. This is particularly useful when you need to create images that run on different hardware architectures, such as ARM-based processors.

- `--load`: This option tells Docker to load the built image into the local Docker image store after the build is complete. This is useful for testing the image locally or for further development.

- `-t rcore_lab`: The `-t` flag is used to tag the image with a name. In this case, the image is tagged as `rcore_lab`. Tagging helps in identifying and managing Docker images, especially when you have multiple images in your local repository.

Putting it all together, this command builds a Docker image for the ARM64 architecture on a Linux platform, loads the image into the local Docker image store, and tags it as `rcore_lab`. This is a powerful way to ensure that your Docker images are compatible with different hardware architectures and can be easily managed and tested locally.

## Run docker container
```
docker run -itd -v $(pwd):/mnt -w /mnt --name rcore_lab_container rcore_lab bash
```
### Explanation

- `-itd`: This combination of flags runs the container in an interactive mode (-i), allocates a pseudo-TTY (-t), and runs the container in detached mode (-d), meaning it runs in the background.
- `-v $(pwd):/mnt`: This option mounts the current working directory ($(pwd)) on the host machine to the /mnt directory inside the container. This allows the container to access and modify files in the host's current directory.
- `-w /mnt`: This sets the working directory inside the container to /mnt, which is where the host's current directory is mounted.
- `--name rcore_lab_container`: This assigns a name (rcore_lab_container) to the container, making it easier to reference in subsequent Docker commands.
- `rcore_lab`: This specifies the Docker image to use for creating the container. In this case, it is an image named rcore_lab.
- `bash` runs the Bash shell inside the container, allowing for interactive command execution.

Overall, this command is useful for setting up a Docker container that has access to the host's current directory, runs in the background, and provides an interactive shell for further operations.


## Pro-tips

Use VSCode + Remote-Container extension to do the development and it  feels like working in a native Ubuntu environment.