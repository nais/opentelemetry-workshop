# Exercise 2: Local setup

In this section, we will set up the environment for the workshop. We will use Docker and Docker Compose to run the services required for the workshop.

## Prerequisites

Make sure you have the following tools installed on your machine:

* [Git](https://git-scm.com/)
* [Docker](https://www.docker.com/)
* [Docker Compose](https://docs.docker.com/compose/)

## Clone the repository

Clone the repository to your local machine:

```bash
git clone git@github.com:nais/opentelemetry-workshop.git
```

## Pull the Docker images

Pull the Docker images required for the workshop:

```bash
docker-compose pull
```

!!! note

    The images are quite large, so it might take a while to download them :coffee:

## Start the application

Start the services required for the workshop:

```bash
docker-compose up -d flagd
docker-compose up -d otel-lgtm
docker-compose up
```

!!! note

    We are phasing the startup to eliminate failures due to dependencies not being ready. The `flagd` service is a simple service that returns a flag. The `otel-lgtm` service is the Grafana LGTM stack.

### Problems starting otel-lgtm?

If you get problems related to volumes, you can try do the following to relace the otel-lgtm service with the following service:

```yaml
  otel-lgtm:
    build:
      context: ./src/otel-lgtm
      dockerfile: Dockerfile
    container_name: otel-lgtm
    restart: unless-stopped
    ports:
      - "${GRAFANA_SERVICE_PORT}"
    environment:
      - GF_SERVER_DOMAIN=localhost
      - GF_SERVER_ROOT_URL=%(protocol)s://%(domain)s/grafana/
      - GF_SERVER_SERVE_FROM_SUB_PATH=true
    logging: *logging
```

And comment out the grafana volume:

```yaml
#volumes:
#  grafana-data:
```

And then build and start the otel-lgtm service:

```bash
docker-compose up -d --build otel-lgtm
```

Verify that the otel-lgtm service is running correctly by executing the following command:

```bash
docker-compose logs otel-lgtm -f
```

It should output something like this:

```bash
otel-lgtm  | Waiting for the OpenTelemetry collector and the Grafana LGTM stack to start up...
otel-lgtm  | The OpenTelemetry collector and the Grafana LGTM stack are up and running.
otel-lgtm  | Open ports:
otel-lgtm  |  - 4317: OpenTelemetry GRPC endpoint
otel-lgtm  |  - 4318: OpenTelemetry HTTP endpoint
otel-lgtm  |  - 3000: Grafana. User: admin, password: admin
```

## Verify the services

Open your browser and navigate to the following URLs:

* http://localhost:8080

## Next steps

Now that you have the repository cloned and the Docker images pulled, you are ready to start the workshop. In the next section, we will cover the basics of instrumentation with OpenTelemetry.

Continue to [Exercise 3: Instrumentation with JAVA](./03-instrumentation.md)