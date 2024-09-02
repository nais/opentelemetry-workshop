# Exerice 6: diagnosing a memory leak

Now it is time to put your skills to the test. In this exercise, you will diagnose a memory leak in a distributed system. You will use OpenTelemetry to collect telemetry data and use it to identify the root cause of the memory leak.

Application telemetry, such as the kind that OpenTelemetry can provide, is very useful for diagnosing issues in a distributed system. In this scenario, we will walk through a scenario demonstrating how to move from high-level metrics and traces to determine the cause of a memory leak.

## Setup

To run this scenario, you will need to have our demo application running and the `recommendationServiceCacheFailure` feature flag enabled. This is enabled by opening the `src/flagd/demo.flagd.json` file and setting the `defaultVariant` to `on` like so:

```json
"recommendationServiceCacheFailure": {
  "description": "Fail recommendation service cache",
  "state": "ENABLED",
  "variants": {
    "on": true,
    "off": false
  },
  "defaultVariant": "off"
}
```

Save the file and let the application run for a couple of minutes to allow for data to populate.

## Diagnosis

The first step in diagnosing a problem is to determine that a problem exists. Often the first stop will be a the metrics dashboard that we created in the previous exercises. I told you some of those metrics would come in handy one day, didn't I?!

Open the Grafana dashboard and look for any anomalies in the metrics. You should see that the recommendation service is experiencing high CPU and memory usage.

![Grafana dashboard](https://opentelemetry.io/docs/demo/scenarios/recommendation-cache/grafana-dashboard.png)

Recommendation Service charts are generated from OpenTelemetry Metrics exported to Prometheus, while the Service Latency and Error Rate charts are generated through the Span Metrics processor.

From our dashboard, we can see that there seems to be anomalous behavior in the recommendation service – spiky CPU utilization, as well as long tail latency in our p95, 99, and 99.9 histograms. We can also see that there are intermittent spikes in the memory utilization of this service.

We know that we’re emitting trace data from our application as well, so let’s think about another way that we’d be able to determine that a problem exists.

## Grafana Tempo

Grafana Tempo allows us to search for traces and display the end-to-end latency of an entire request with visibility into each individual part of the overall request. Perhaps we noticed an increase in tail latency on our frontend requests. Tempo allows us to then search and filter our traces to include only those that include requests to the recommendation service.


By sorting by latency, we’re able to quickly find specific traces that took a long time. Clicking on a trace we’re able to view the waterfall view.

We can see that the recommendation service is taking a long time to complete its work, and viewing the details allows us to get a better idea of what’s going on.

## Confirming the Diagnosis

Inspect the trace details for a trace that took a long time to complete.

!!! note

    :question: Can you spot any anomalies in the trace details such as span attributes or tags that might indicate the root cause of the memory leak?

    <details>
    <summary>Hint</summary>

    We can see in our trace details view that the `app.cache_hit` attribute is set to `false`, and that the `app.products.count` value is extremely high.
    </details>

    :question: Can you find previous traces that exhibit the same or different behavior?

    <details>
    <summary>Hint</summary>

    Try filter for `recommendationservice` in the service name, and search for `app.cache_hit=true` in tag search. Notice that requests tend to be faster when the cache is hit. Now search for `app.cache_hit=false` and compare the latency. You should notice some changes in the visualization at the top of the trace list.

## Conclusion

Now, since this is a contrived scenario, we know where to find the underlying bug in our code. However, in a real-world scenario, we may need to perform further searching to find out what’s going on in our code, or the interactions between services that cause it.

In this scenario, we were able to use OpenTelemetry to diagnose a memory leak in our recommendation service. We were able to use metrics to identify that there was a problem, and traces to confirm our suspicions and find the root cause of the issue.

Continue to [Exercise 7: More challenges](./07-challenge-2.md) to put your skills to the test and diagnose even more things that can go wrong in a distributed system.
