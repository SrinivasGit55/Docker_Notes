# Docker Day 4 — Docker Compose

## What Is Docker Compose?

Docker Compose is a tool for defining and running a multi-container application using a YAML configuration file. It lets us describe services, networks, volumes, environment settings, and other application resources in one place, then manage them with Compose commands.

Compose is useful when an application needs multiple containers on one Docker host, such as a web frontend, API, and database.

## Deployment Without Compose

Running each container separately requires repeated commands, manual network configuration, and remembering options. As the application grows, managing the containers becomes harder.

Compose stores the configuration declaratively in a file, so the environment is easier to reproduce and maintain.

## Compose File

A Compose file is commonly named `compose.yaml` or `docker-compose.yml`. Current Compose uses the Compose Specification; the old top-level `version:` field is generally unnecessary.

Example web service:

```yaml
services:
  web:
    build: .
    image: webapp:v1
    ports:
      - "8080:80"
```

The `services` section defines containers to run. `build` points to a build context, `image` gives the resulting image name, and `ports` maps host port 8080 to container port 80.

## Multi-Service Example

```yaml
services:
  web:
    build: .
    image: webapp:v1
    ports:
      - "8080:80"
    depends_on:
      - redis
  redis:
    image: redis:7
```

This example defines two services. Compose creates a default network and allows the services to communicate by service name, so the application can reach Redis using the hostname `redis`.

`depends_on` controls startup ordering, but it does not by itself guarantee that a dependency is healthy and ready to accept requests.

## Common Compose Commands

Start services in the foreground:

```bash
docker compose up
```

Start in detached mode:

```bash
docker compose up -d
```

Build images as part of startup:

```bash
docker compose up --build -d
```

List services and containers:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs
docker compose logs -f web
```

Stop and remove the containers and default network:

```bash
docker compose down
```

Remove containers and associated named/anonymous volumes declared for the project (use caution):

```bash
docker compose down --volumes
```

## Scaling a Service

Compose can run multiple container instances of a service on a single host:

```bash
docker compose up -d --scale web=3
```

Scaling may be limited when the service publishes a fixed host port, because multiple instances cannot all bind the same host port in the same way. Use a load balancer or suitable routing design for access to multiple replicas.

## Compose and Docker Swarm

Docker Compose is mainly used to manage a multi-container application on a single Docker host. Compose does not schedule containers across Swarm nodes with `docker compose up`.

For multi-node Swarm deployment, use Swarm services and `docker stack deploy` with a compatible Compose file. The Swarm stack command has compatibility constraints and does not support every modern Compose feature.

## Example Workflow

```text
Source code and Dockerfiles
        |
        v
compose.yaml
        |
        v
docker compose up --build -d
        |
        +--> Web container
        +--> API container
        +--> Database container
        |
        v
docker compose ps / logs
```

## Practical DevOps Use

Compose is useful for local development, test environments, integration testing, and small single-host deployments. It can define the same application dependencies consistently across team members' machines and CI jobs. For production, evaluate availability, security, storage, secrets, backups, monitoring, and whether orchestration beyond a single host is required.

## Key Takeaways

- Compose describes an application made of multiple services in YAML.
- `docker compose up` creates and starts the services defined in the file.
- Compose manages project networks and declared volumes.
- Service-to-service communication commonly uses service names.
- Compose scaling is on a single host; it is not multi-node scheduling.
- Docker Swarm uses `docker service` and `docker stack deploy` for cluster deployment.
