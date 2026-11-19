# Docker Day 2 — Dockerfile, Images and Build Instructions

## Docker Images and Container Changes

A Docker image is a reusable, read-only template. A container is an instance created from an image, with its own writable layer. Changes made manually inside a container are not automatically part of the original image.

```bash
docker pull ubuntu
docker images
docker run -it --name container1 ubuntu
```

Inside the container, package installation and file changes modify its writable layer:

```bash
apt update
apt install git maven apache2 tree -y
touch file1
```

Detach without stopping the container using `Ctrl+P`, then `Ctrl+Q`. Reattach with:

```bash
docker attach container1
```

## Commit a Container into an Image

`docker commit` creates an image from a container's current filesystem state. It can be useful for learning or quick snapshots, but a Dockerfile is preferable for repeatable builds and reviewable changes.

```bash
docker commit container1 raham:v1
docker images
docker run -it --name container2 raham:v1
```

## Dockerfile

A Dockerfile is a text file containing instructions Docker uses to build an image. It automates image creation instead of manually configuring each container.

```dockerfile
FROM ubuntu
RUN apt-get update && apt-get install -y git maven tree apache2
```

Build and run:

```bash
docker build -t netflix:v1 .
docker run -it --name container3 netflix:v1
```

The final `.` is the build context: Docker sends files from this directory (subject to ignore rules) to the builder.

## Common Dockerfile Instructions

### FROM
Defines the base image.

```dockerfile
FROM ubuntu
```

### RUN
Executes commands during image build, such as installing packages.

```dockerfile
RUN apt-get update && apt-get install -y git maven tree
```

### CMD
Defines the default command or arguments used when a container starts. It is not intended for build-time package installation.

```dockerfile
CMD ["bash"]
```

### COPY
Copies files from the build context into the image.

```dockerfile
COPY index.html /tmp/
```

### ADD
Can copy local files and supports additional behaviors such as archive extraction and remote URL handling. Use `COPY` for ordinary local file copying because its intent is clearer.

### WORKDIR
Sets the working directory for later Dockerfile instructions and the default working directory at runtime.

```dockerfile
WORKDIR /tmp
```

### LABEL
Adds metadata to an image.

```dockerfile
LABEL author="Your Name"
```

Inspect labels and image metadata:

```bash
docker inspect netflix:v1
```

### ENV
Sets environment variables that persist in the image and are available to containers.

```dockerfile
ENV name="Vijay"
ENV client="Swiggy"
```

### ARG
Defines build-time variables. They are not a safe place for secrets.

```dockerfile
ARG APP_VERSION=1.0
RUN echo "Building version $APP_VERSION"
```

```bash
docker build --build-arg APP_VERSION=2.0 -t app:v2 .
```

### ENTRYPOINT and CMD
`ENTRYPOINT` defines the executable, while `CMD` can provide default arguments. They can be used together.

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello from Docker"]
```

```bash
docker run myimage
docker run myimage "Hello DevOps"
```

### VOLUME
Declares a mount point intended for persistent or externally managed data. The declaration alone is not a backup strategy.

```dockerfile
VOLUME /data
```

### EXPOSE
Documents the port the application is expected to listen on. It does not publish the port to the host by itself.

```dockerfile
EXPOSE 8080
docker run -d -p 8080:8080 myapp:v1
```

## Build Context and .dockerignore

The build context is the directory or set of files available to the image build. Exclude unnecessary files using a `.dockerignore` file to reduce transfer size and avoid copying secrets or irrelevant files.

Example:

```
.git
node_modules
*.log
.env
```

## Cleanup (Use Carefully)

The original bulk cleanup pattern can remove every container or image, including resources you need. Prefer checking and removing named resources.

```bash
docker ps -a
docker stop container1
docker rm container1
docker images
docker rmi netflix:v1
```

## Practical Build Example

```dockerfile
FROM ubuntu
RUN apt-get update && apt-get install -y apache2 && rm -rf /var/lib/apt/lists/*
COPY index.html /var/www/html/index.html
EXPOSE 80
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

```bash
docker build -t webapp:v1 .
docker run -d --name webapp -p 8080:80 webapp:v1
```

## Key Takeaways

- Use Dockerfiles for repeatable image creation.
- Use `RUN` for build-time setup and `CMD` for default runtime behavior.
- Prefer `COPY` for ordinary file copies.
- `ARG` is build-time configuration; `ENV` is available at runtime.
- `EXPOSE` is metadata; publish ports with `-p`.
- Do not place credentials in image layers or build arguments.
- Container commit is useful as a snapshot but is less reproducible than a Dockerfile.
