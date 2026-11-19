# Docker Day 5 — Docker Swarm, Services and Scaling

## Docker Swarm

Docker Swarm is Docker Engine's native clustering and orchestration mode. It allows multiple Docker hosts to work together as a cluster and lets a manager schedule service tasks across available nodes.

Swarm has manager and worker roles. Managers maintain cluster state and schedule tasks; worker nodes run tasks assigned by the manager. A manager can also run workloads unless configured otherwise.

## Swarm Nodes

- **Manager node:** Maintains cluster state, handles orchestration and scheduling, and accepts Swarm management commands.
- **Worker node:** Receives and runs service tasks from the manager.

For a lab, three servers can be used: one manager and two workers. In production, design manager availability, networking, security, and capacity carefully.

## Initialize a Swarm

On the node intended to be the manager:

```bash
docker swarm init
```

The command returns a join command and token for worker nodes. Run the provided join command on each worker. Do not share join tokens publicly.

Check node membership on the manager:

```bash
docker node ls
```

## Docker Hub and Image Distribution

Swarm nodes need access to the image used by service tasks. Build the application image and publish it to a registry so other nodes can pull it.

```bash
docker build -t myapp:v1 .
docker login
docker tag myapp:v1 YOUR_DOCKERHUB_USERNAME/myapp:v1
docker push YOUR_DOCKERHUB_USERNAME/myapp:v1
```

Replace the placeholder with your Docker Hub username and use secure authentication. Avoid embedding credentials in commands, scripts, or source files.

## Swarm Services

A service describes the desired state of an application running in the Swarm: image, number of tasks, ports, networks, constraints, and resource settings. Swarm continuously attempts to maintain the desired task count.

Create a service with three replicas:

```bash
docker service create --name movies --replicas 3 -p 8080:80 YOUR_DOCKERHUB_USERNAME/myapp:v1
```

List services:

```bash
docker service ls
```

Inspect service details:

```bash
docker service inspect movies
```

List tasks/containers belonging to a service:

```bash
docker service ps movies
```

## Scaling a Service

Change the desired replica count:

```bash
docker service scale movies=6
docker service ls
docker service ps movies
```

Scaling up increases the desired task count; scaling down decreases it. Swarm places tasks according to available resources and scheduling constraints.

## Self-Healing Behavior

If a service task/container fails or is removed, Swarm attempts to replace it to maintain the desired replica count. This is orchestration behavior, not a guarantee that the application is healthy or that data is recovered.

A container created manually with `docker run` is not automatically managed as a Swarm service and does not receive the same desired-state replacement behavior.

## Node Failure and Scheduling

Swarm can reschedule service tasks when nodes become unavailable, provided there are healthy nodes with sufficient resources and the required image, network, storage, and constraints. A task may be recreated on another node, so local-only data and bind-mounted paths require careful planning.

## Remove a Service

```bash
docker service rm movies
```

## Swarm Command Reference

```bash
docker swarm init
docker swarm join --token <token> <manager-ip>:2377
docker node ls
docker service ls
docker service ps <service>
docker service inspect <service>
docker service scale <service>=<replicas>
docker service rm <service>
```

## Key Takeaways

- Swarm orchestrates containers across multiple Docker hosts.
- Services define desired state and replica count; Swarm creates tasks to match.
- Images should be available through a registry for nodes that need to pull them.
- Scaling changes the desired number of tasks, not just manually started containers.
- Swarm can replace failed tasks, but does not guarantee application-level health, data durability, or full high availability.
