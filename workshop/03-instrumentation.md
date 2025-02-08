# Exercise 3: Instrumentation with JAVA

In this exercise, we will instrument the `ad` service with OpenTelemetry. We will use the OpenTelemetry Java Agent to auto-instrument the application before adding custom traces and metrics.

If you want to learn more about the OpenTelemetry Java Instrumentation, you can read the documentation [github.com/open-telemetry/opentelemetry-java-instrumentation](https://github.com/open-telemetry/opentelemetry-java-instrumentation/)

### Assignment 1 - Configure OpenTelemetry agent

First we need to add the necessary configuration for the OpenTelemetry agent so that it can connect to the OpenTelemetry collector.

You need to configure the following environment variables in the `ad` service in the [`docker-compose.yml`](../docker-compose.yml) file. Look for the `# @TODO add otel env vars here` comment in the `docker-compose.yml` file:

```yaml
  # AdService
  ad:
    ...
    environment:
      ...
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://${OTEL_COLLECTOR_HOST}:${OTEL_COLLECTOR_PORT_HTTP}
      - OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE
      - OTEL_RESOURCE_ATTRIBUTES
      - OTEL_LOGS_EXPORTER=otlp
      - OTEL_SERVICE_NAME=ad
```

<details>
<summary>1. What is the purpose of the `OTEL_EXPORTER_OTLP_ENDPOINT` environment variable?</summary>

The `OTEL_EXPORTER_OTLP_ENDPOINT` environment variable specifies the endpoint where the OpenTelemetry collector is running. It is used by the OpenTelemetry agent to send collected telemetry data such as traces and metrics.

</details>

<details>
<summary>2. What does the `OTEL_SERVICE_NAME` environment variable define?</summary>

The `OTEL_SERVICE_NAME` environment variable defines the name of the service being instrumented. This name is used to identify the service in telemetry data, making it easier to analyze and monitor.

</details>

<details>
<summary>3. How does the `OTEL_LOGS_EXPORTER` environment variable affect logging?</summary>

The `OTEL_LOGS_EXPORTER` environment variable specifies the exporter to be used for sending logs. Setting it to `otlp` means that logs will be sent to the OpenTelemetry collector using the OTLP protocol.

</details>
<br />

Next we need to add the Java agent to the `ad` service. We do this by downloading the Java agent and adding it to the `JAVA_TOOL_OPTIONS` environment variable inside the [`ad` Dockerfile](../src/ad/Dockerfile).

In the final build step of the `ad` Dockerfile, you need to add the following lines. Look for the `# @TODO add otel java agent here` comment in the Dockerfile:

```dockerfile
...
ADD --chmod=644 https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v$OTEL_JAVA_AGENT_VERSION/opentelemetry-javaagent.jar /usr/src/app/opentelemetry-javaagent.jar
ENV JAVA_TOOL_OPTIONS=-javaagent:/usr/src/app/opentelemetry-javaagent.jar
...
```

<details>
<summary>4. What is the purpose of the `ADD` statement in the Dockerfile?</summary>

The `ADD` statement in the Dockerfile is used to download the OpenTelemetry Java agent from the specified URL and add it to the `/usr/src/app/` directory inside the Docker image. This allows the Java application to use the agent for instrumentation.

</details>

<details>
<summary>5. How does the `JAVA_TOOL_OPTIONS` environment variable affect the Java application?</summary>

The `JAVA_TOOL_OPTIONS` environment variable is used to pass options to the Java Virtual Machine (JVM). By setting it to `-javaagent:/usr/src/app/opentelemetry-javaagent.jar`, it tells the JVM to use the OpenTelemetry Java agent for instrumentation when running the Java application.

</details>
<br />

Now you can rebuild the `ad` service with the following command:

```bash
docker-compose down ad
docker-compose up ad --build -d
```

You should be able to get standard metrics and traces from the ad. Open [Grafana](http://localhost:8080/grafana) and navigate to Explore and select Tempo as the datasource, click on the Service Ggraph. Adservice should now be available there.

### Assignment 2 - Add attributes and events

We can use the span context to hook into so we can enrich traces with more information.  Read about how it's done : <https://opentelemetry.io/docs/languages/java/instrumentation/>

__Assignment  :__

In get `getAds` method hook into the span context and add the following attributes to the span `app.ads.contextKeys`, `app.ads.contextKeys.count`, `app.ads.count`, `app.ads.ad_request_type`, `app.ads.ad_response_type` .
Data that you are adding to the attributes are defined in the demo.proto file. This will give us insight into what advertisement that has been shown.

You can rebuild the `ad` with the following command :

```bash
docker-compose down ad
docker-compose up ad --build -d
```

You can verify that the attributes are set by looking at the traces in Grafana with Tempo as the datasource.

#### Span Events

A Span Event can be thought of as a structured log message (or annotation) on a Span, typically used to denote a meaningful, singular point in time during the Span’s duration.

For example, consider two scenarios in a web browser:

* Tracking a page load
* Denoting when a page becomes interactive
A Span is best used to the first scenario because it’s an operation with a start and an end.

A Span Event is best used to track the second scenario because it represents a meaningful, singular point in time.

#### When to use span events versus span attributes

Since span events also contain attributes, the question of when to use events instead of attributes might not always have an obvious answer. To inform your decision, consider whether a specific timestamp is meaningful.

For example, when you’re tracking an operation with a span and the operation completes, you might want to add data from the operation to your telemetry.

* If the timestamp in which the operation completes is meaningful or relevant, attach the data to a span event.
* If the timestamp isn’t meaningful, attach the data as span attributes.

__Assignment  :__
Next we want to refine the traces from the span with events and status codes if getAds method fails. Inside the `try/catch` block add and error event with attributeKey thats should be called : `exception.message` that adds the exception to the span. We should also as part of this mark the span as failed.

### Assignment.3 - Create new spans

#### Spans

A span represents a unit of work or operation. Spans are the building blocks of Traces. In OpenTelemetry, they include the following information:

* Name
* Parent span ID (empty for root spans)
* Start and End Timestamps
* Span Context
* Attributes
* Span Events
* Span Links
* Span Status

Read more here : <https://opentelemetry.io/docs/concepts/signals/traces/#spans>

__Assignment  :__
In the Adservice we would like to add a span for just `getRandomAds()` method and add it to the tracing context.

### Assignment.4 - Add Metrics

A metric is a measurement of a service captured at runtime. The moment of capturing a measurements is known as a metric event,
which consists not only of the measurement itself, but also the time at which it was captured and associated metadata.
Application and request metrics are important indicators of availability and performance.
Custom metrics can provide insights into how availability indicators impact user experience or the business.
Collected data can be used to alert of an outage or trigger scheduling decisions to scale up a deployment automatically upon high demand.

Read more about  metrics here :  <https://opentelemetry.io/docs/concepts/signals/metrics/>

Assignment :
Create metrics that counts requests by request and response type

Also make an adRequestsCounter with the span metrics called : `app.ads.count`, `app.ads.ad_request_type`, `app.ads.ad_response_type`

### Assignment.5 - Add session_id as Baggage

Baggage in OpenTelemetry is a mechanism for propagating context across process boundaries in distributed systems. It allows you to attach arbitrary key-value pairs to a request
and have them travel along with that request as it moves through different services or components. Unlike spans, which are primarily used for tracing and measuring the execution of operations,
baggage is intended to carry additional contextual information that might be relevant for various parts of your system.

__Assignment  :__
You will be modifying a service to extract a session.id from the current context's baggage. If the session.id is present,
it should be used to enrich the current span and to update a custom context object (evaluationContext).
If no baggage is found, you will handle this case by logging an appropriate message.

### Assignment.6 - Add logs

OpenTelemetry does not define a bespoke API or SDK to create logs. Instead, OpenTelemetry logs are the existing logs you already have from a logging framework or infrastructure component. OpenTelemetry SDKs and autoinstrumentation utilize several components to automatically correlate logs with traces.

OpenTelemetry’s support for logs is designed to be fully compatible with what you already have, providing capabilities to wrap those logs with additional context and a common toolkit to parse and manipulate logs into a common format across many different sources.

In our instance we are using Log4j that automaticly sends logs to the OpenTelemetry collector.

Read more here :  <https://opentelemetry.io/docs/concepts/signals/logs/>

## Skipping the assignment

If you are having trouble with the assignment, you can skip it and continue with the next one by changing the `docker-compose.yml` to use the pre-built image for the `ad` like this and commenting out the build section:

```yaml
  ad:
    image: ${IMAGE_NAME}:${DEMO_VERSION}-ad
    container_name: ad-service
    #build:
    #  context: ./
    #  dockerfile: ${AD_SERVICE_DOCKERFILE}
    #  cache_from:
    #    - ${IMAGE_NAME}:${IMAGE_VERSION}-ad
```

And setting the environments variables like this:

```yaml
    environment:
      - AD_SERVICE_PORT
      - FLAGD_HOST
      - FLAGD_PORT
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://${OTEL_COLLECTOR_HOST}:${OTEL_COLLECTOR_PORT_HTTP}
      - OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE
      - OTEL_RESOURCE_ATTRIBUTES
      - OTEL_LOGS_EXPORTER=otlp
      - OTEL_SERVICE_NAME=ad
```
