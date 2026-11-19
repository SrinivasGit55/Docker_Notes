# Docker Day 7 — Portainer, Networking, Resource Limits and JFrog

## Portainer

Portainer is a web-based management interface for container environments. It provides a graphical way to inspect and manage containers, images, volumes, networks, services, and Swarm resources. It is an additional application, not a built-in Docker CLI command.

Portainer components and deployment options depend on the environment. Portainer Server manages the UI and environment connections; Portainer Agent can be deployed to nodes to provide access to remote Docker environments.

### Portainer in a Swarm Lab

The lab uses Swarm and a stack file to deploy Portainer-related components. Swarm stack deployment is performed from a manager node.

Typical commands for stack inspection:

```bash
docker stack ls
docker stack services <stack-name>
docker stack ps <stack-name>
docker service ls
```

The stack file must match the deployment mode and Portainer version. Use the current Portainer deployment instructions for the exact Server/Agent configuration, ports, volumes, and image tags rather than copying an old stack file blindly.

## Docker Networking

Docker networks provide connectivity between containers and define how containers reach each other and external networks.

List networks:

```bash
docker network ls
```

Inspect a network:

```bash
docker network inspect bridge
```

Create a user-defined bridge network:

```bash
docker network create paytm
docker network inspect paytm
```

Start containers attached to a network:

```bash
docker run -dit --name container1 --network paytm ubuntu
docker run -dit --name container2 --network paytm ubuntu
```

Connect an existing container to a network:

```bash
docker network connect paytm container1
```

Disconnect a container:

```bash
docker network disconnect paytm container1
```

Remove a network when it is no longer in use:

```bash
docker network rm paytm
```

A user-defined bridge network allows containers on that network to communicate and provides name-based service discovery. Docker's default bridge has different behavior; user-defined networks are generally preferred for application connectivity.

## Container Resource Limits

Containers share host resources. Without resource constraints, a container may consume more CPU or memory than expected and affect other workloads. Resource limits help control resource usage.

### CPU Limit

Limit a container to approximately half of one CPU:

```bash
docker run -d --name limited-app --cpus="0.5" nginx
```

### Memory Limit

Set a memory limit:

```bash
docker run -d --name limited-memory --memory="512m" nginx
```

Combined example:

```bash
docker run -d --name app-limited --cpus="0.5" --memory="512m" nginx
```

Limits should be based on observed application needs. Very low limits can cause performance issues, restarts, or out-of-memory termination.

## Monitor Resource Usage

View live container resource usage:

```bash
docker stats
docker stats container1
```

Inspect configured limits and runtime settings:

```bash
docker inspect container1
```

`docker stats` shows live CPU, memory, network, and block I/O statistics. `docker inspect` provides configuration details, including resource settings.

## JFrog Artifactory

JFrog Artifactory is a repository manager used to store and manage build artifacts and packages. It supports multiple package formats and can also be configured as a Docker/container registry.

In a DevOps pipeline, a repository manager can help centralize artifacts produced by builds, manage versions, control access, and support promotion between development, testing, and release stages.

### Typical CI/CD Flow

```text
Developer commits source
        |
        v
      Jenkins
        |
        v
Build and test
        |
        v
Publish artifact / image
        |
        v
JFrog Artifactory or container registry
        |
        v
Deployment environment pulls approved version
```

Artifactory is not the same thing as Docker Engine or Docker Swarm. It is a repository platform; Docker hosts and orchestrators consume images or artifacts stored in repositories.

### Artifactory in a Container Lab

Artifactory can be deployed using Docker, but it is a stateful, resource-intensive service. The installation must include persistent storage and appropriate CPU/memory sizing. Use the current official JFrog installation guide and a supported version for the exact container command or Compose configuration. Do not use an unpinned or obsolete image tag for a long-lived deployment.

## Resource and Cost Awareness

When running AWS EC2 Docker labs:

- Stop instances when the lab is not in use.
- Swarm managers, Jenkins, Portainer, and Artifactory consume memory and CPU.
- Verify instance type and storage requirements before deploying.
- Review containers, services, volumes, and images before deleting resources.
- Do not assume Docker resource limits reduce the EC2 instance's billed size.

## Key Takeaways

- Portainer is a GUI for Docker management; it complements CLI tools.
- Networks control container connectivity; custom networks support service-name communication.
- Resource limits protect the host from one container consuming excessive resources.
- `docker stats` is useful for live resource monitoring.
- Artifactory is a repository manager that can store build artifacts and container images.
- Stateful services need persistent storage, adequate resources, backups, and a maintenance plan.
