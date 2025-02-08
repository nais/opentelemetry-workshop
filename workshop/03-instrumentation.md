# Exercise 3: Instrumentation with Java

In this exercise, the `ad` service will be instrumented with OpenTelemetry using the OpenTelemetry Java Agent. The goal is to configure automatic instrumentation before adding custom traces, metrics, logs, and baggage support.

If you want more details about OpenTelemetry Java Instrumentation, see the documentation: [github.com/open-telemetry/opentelemetry-java-instrumentation](https://github.com/open-telemetry/opentelemetry-java-instrumentation/)

---

## Task 1 – Configure the OpenTelemetry Agent

To connect the OpenTelemetry agent to the collector, add the following environment variables to the `ad` service in the [`docker-compose.yml`](../docker-compose.yml) file. Look for `# @TODO add otel env vars here`:

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
- Points to the collector endpoint for traces, metrics, and logs
- Exports logs with the OTLP exporter
- Names the service as `ad`

Next, add the Java agent to the `ad` service by downloading and referencing it in the `JAVA_TOOL_OPTIONS` environment variable inside the [`ad` Dockerfile](../src/ad/Dockerfile). Look for `# @TODO add otel java agent here`:

```dockerfile
...
ADD --chmod=644 https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v$OTEL_JAVA_AGENT_VERSION/opentelemetry-javaagent.jar /usr/src/app/opentelemetry-javaagent.jar
ENV JAVA_TOOL_OPTIONS=-javaagent:/usr/src/app/opentelemetry-javaagent.jar
...
```
- Downloads the agent from GitHub
- Places it under `/usr/src/app/`
- Uses `JAVA_TOOL_OPTIONS` to start the JVM with the agent

Rebuild the `ad` service:

```bash
docker-compose down ad
docker-compose up ad --build -d
```

---

## Task 2 – Add Attributes and Events

To refine trace analysis, add custom attributes and events to spans in the `ad` service’s [`AdService.java`](../src/ad/src/main/java/oteldemo/AdService.java) file.

1. Get the current span (look for `// @TODO: get the current span in context`):

```java
Span currentSpan = Span.current();
```
- Captures the active span
- Lets you modify span data
- Reflects the ongoing trace context

2. Set custom attributes (replace the `// @TODO: set the span attributes` comments accordingly):

```java
span.setAttribute("app.ads.contextKeys", req.getContextKeysList().toString());
span.setAttribute("app.ads.contextKeys.count", req.getContextKeysCount());
...
span.setAttribute("app.ads.count", allAds.size());
span.setAttribute("app.ads.ad_request_type", adRequestType.name());
span.setAttribute("app.ads.ad_response_type", adResponseType.name());
```
- Adds contextual data to the current span
- Associates details about the request and response
- Helps with trace-based troubleshooting

3. Add an error event in the `try/catch` block if `getAds` fails (replace `// @TODO: add span event`):

```java
span.addEvent(
  "Error", Attributes.of(AttributeKey.stringKey("exception.message"), e.getMessage()));
span.setStatus(StatusCode.ERROR);
```
- Logs an error event in the span
- Stores exception details
- Marks the span as failed

Rebuild the `ad` service:

```bash
docker-compose down ad
docker-compose up ad --build -d
```

---

## Task 3 – Create New Spans

Spans represent individual operations within a trace. In `AdService.java`, create new spans for `getAdsByCategory` and `getRandomAds`:

Replace `// @TODO: create a new span for getAdsByCategory`:

```java
@WithSpan("getAdsByCategory")
private Collection<Ad> getAdsByCategory(@SpanAttribute("app.ads.category") String category) {
  Collection<Ad> ads = adsMap.get(category);
  Span.current().setAttribute("app.ads.count", ads.size());
  return ads;
}
```
- Creates a named span for category-based lookups
- Adds an attribute for ads count
- Encapsulates category handling in a separate span

Replace `// @TODO: create a new span for getRandomAds`:

```java
private List<Ad> getRandomAds() {
  List<Ad> ads = new ArrayList<>(MAX_ADS_TO_SERVE);
  Span span = tracer.spanBuilder("getRandomAds").startSpan();
  try (Scope ignored = span.makeCurrent()) {

    Collection<Ad> allAds = adsMap.values();
    for (int i = 0; i < MAX_ADS_TO_SERVE; i++) {
      ads.add(Iterables.get(allAds, random.nextInt(allAds.size())));
    }
    span.setAttribute("app.ads.count", ads.size());

  } finally {
    span.end();
  }
  return ads;
}
```
- Manually starts a new span for random ad selection
- Tracks random picks inside the span
- Ends the span after finishing the operation

Rebuild `ad`:

```bash
docker-compose down ad
docker-compose up ad --build -d
```

---

## Task 4 – Add Metrics

OpenTelemetry metrics measure a service at runtime. To track request counts and types, add custom metric counters in `AdService.java`. Replace `// @TODO: add metric counter` with:

```java
private static final LongCounter adRequestsCounter =
  meter
    .counterBuilder("app.ads.ad_requests")
    .setDescription("Counts ad requests by request and response type")
    .build();

private static final AttributeKey<String> adRequestTypeKey =
  AttributeKey.stringKey("app.ads.ad_request_type");
private static final AttributeKey<String> adResponseTypeKey =
  AttributeKey.stringKey("app.ads.ad_response_type");
```
- Defines a counter for ad requests
- Creates attribute keys for request and response types
- Helps monitor the overall request flow

Then increment the counter (replace `// @TODO: count the number of ad requests`):

```java
adRequestsCounter.add(
  1,
  Attributes.of(
    adRequestTypeKey, adRequestType.name(), adResponseTypeKey, adResponseType.name()));
```
- Increments the counter each time an ad is requested
- Tags each metric with request and response types
- Enables deeper analysis of request patterns

Rebuild `ad`:

```bash
docker-compose down ad
docker-compose up ad --build -d
```

---

## Task 5 – Extract Baggage

Baggage helps propagate context across distributed services. Extract a `session.id` from baggage and add it to the current span. Replace `// @TODO: extract session ID from baggage`:

```java
Baggage baggage = Baggage.fromContextOrNull(Context.current());
if (baggage != null) {
  final String sessionId = baggage.getEntryValue("session.id");
  span.setAttribute("session.id", sessionId);
  evaluationContext.setTargetingKey(sessionId);
  evaluationContext.add("session", sessionId);
} else {
  logger.info("no baggage found in context");
}
```
- Retrieves session ID from baggage
- Sets the `session.id` as a span attribute
- Simplifies passing user session details downstream

Rebuild `ad`:

```bash
docker-compose down ad
docker-compose up ad --build -d
```

---

## Task 6 – Add Logs

OpenTelemetry doesn't define a dedicated logging API; instead, it works with existing logging frameworks. In this workshop, Log4j automatically sends logs to the OpenTelemetry collector. Read more about OpenTelemetry logs here: <https://opentelemetry.io/docs/concepts/signals/logs/>

---

## Verify Instrumentation

After completing the steps, verify the instrumentation by checking the traces, metrics, and logs in Grafana.

## Next steps

Now that you have instrumented the `ad` service with OpenTeTelemetry, you can continue to the next exercise to learn how to analyze the data in Grafana.

Continue to [Exercise 4: Analyzing Data in Grafana](./04-grafana.md)
