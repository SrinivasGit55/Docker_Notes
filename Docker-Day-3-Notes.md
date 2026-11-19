# Docker Day 3 — Volumes, Bind Mounts and Docker System Commands

## Docker Volumes

Docker volumes provide storage that is independent of a container's writable layer. They are useful when application data must persist after a container is removed or be shared by multiple containers.

### Why Volumes Are Needed

Data written only into a container's writable layer is tied to that container. Removing the container removes that layer. Volumes provide a separate storage location managed by Docker.

## Create and Inspect a Volume

```bash
docker volume create volume3
docker volume ls
docker volume inspect volume3
```

## Mount a Named Volume

```bash
docker run -it --name container5 \
  --mount source=volume3,target=/volume3 \
  ubuntu
```

The `source` identifies the named volume and `target` is the mount location inside the container.

The short syntax is also available:

```bash
docker run -it --name container6 -v volume3:/volume3 ubuntu
```

## Share Data Between Containers

Containers can mount the same named volume. Both then access the shared data, subject to filesystem permissions and application behavior.

```bash
docker run -it --name containerA -v shared-data:/data ubuntu
docker run -it --name containerB -v shared-data:/data ubuntu
```

The `--volumes-from` option can also mount volumes from another container:

```bash
docker run -it --name container2 --volumes-from container1 ubuntu
```

This couples the second container to the first container's declared mounts; named volumes are often clearer for independently managed containers.

## Dockerfile VOLUME

```dockerfile
FROM ubuntu
VOLUME /volume1
```

The `VOLUME` instruction declares a mount point. It does not define backup, replication, or a cloud storage service.

## Bind Mounts

A bind mount maps a specific host path into a container. Unlike named volumes, the host directory is selected directly by the user.

```bash
docker run -it --name container7 \
  -v /root:/ABC \
  ubuntu
```

Use bind mounts for development files, configuration, or when the host directory itself must be accessed. Be careful with permissions and with exposing host files to containers.

## Volume Data on the Host

Docker stores local-driver volume data under Docker-managed locations on Linux, commonly within `/var/lib/docker/volumes`. The internal path is implementation-managed. Avoid editing it directly; access the data through the mounted volume instead.

## Docker Disk Usage

```bash
docker system df
docker system df -v
```

These commands report Docker disk usage for images, containers, local volumes, and build cache, with the verbose form showing more detail.

## Prune Unused Resources

```bash
docker system prune
```

This removes unused Docker resources according to the command's scope and prompts for confirmation. Review the prompt carefully. Options such as `-a` and volume-related flags expand what can be removed and may delete data you need.

## Docker Events

```bash
docker system events
```

Displays Docker daemon events such as container, image, network, and volume activity. It is useful for observing what is happening during operations.

## Docker Networks (Preview)

```bash
docker network ls
docker network create my-network
docker network inspect my-network
```

Networks provide connectivity between containers and external systems. Dedicated networking is covered further in the later notes.

## Jenkins Container Introduction

Jenkins can run as a container using its published image. A basic lab example:

```bash
docker run -d --name jenkins -p 8080:8080 jenkins/jenkins:lts
```

For a practical Jenkins deployment, persistent storage, required ports, security, and the appropriate image tag must be considered.

## Key Takeaways

- Volumes persist data independently of a container's writable layer.
- Named volumes are Docker-managed; bind mounts use explicit host paths.
- Sharing a volume allows containers to access the same files, but does not automatically solve concurrent-write or data-consistency issues.
- Use Docker's commands to inspect storage; avoid manually manipulating Docker's internal paths.
- Inspect resources before pruning.
