# Docker Prometheus Demo

This demo sets up Prometheus and Grafana on a Docker Swarm cluster. It monitors [cAdvisor](https://github.com/google/cadvisor) for container metrics, [Node Exporter](https://github.com/prometheus/node_exporter) for host metrics, and the [Docker Engine](https://docs.docker.com/engine/) for Docker-internal metrics.

## Requirements

This demo requires a Swarm cluster to be set up.

## Running the demo

Run our own Docker image registry:

    docker service create --name registry --publish 5000:5000 registry:2

Build custom Prometheus image with our config baked in:

    docker build -t localhost:5000/prometheus .

Push our custom Prometheus image to our registry:

    docker push localhost:5000/prometheus

Deploy Prometheus, Grafana, and the necessary exporters, as a Docker Stack with the name `prometheus`:

    docker stack deploy -c compose.yml prometheus

## Using Prometheus and Grafana

You can reach Prometheus on any host of the cluster on port `9090`.

You can reach Grafana on any host of the cluster on port `3000`. The login is `admin` / `admin`.

## Example queries

For each host, show the total CPU usage in cores:

    sum without(cpu, mode) (rate(node_cpu{mode!="idle"}[1m]))

For each Docker container, show the total CPU usage:

    sum without(cpu) (rate(container_cpu_usage_seconds_total{id=~"/docker/.*"}[1m]))

Swarm-wide, show the per-second rate of all Docker engine container actions.

    sum without(instance) (rate(engine_daemon_container_actions_seconds_count[1m]))
