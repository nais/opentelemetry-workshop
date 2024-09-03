# Exercise 7: More challenges

Out application provides several feature flags that you can use to simulate different scenarios. These flags are managed by flagd, a simple feature flag service that supports OpenFeature. Flag values are stored in the `src/flagd/demo.flagd.json` file. To enable a flag, change the `defaultVariant` value in the config file for a given flag to `"on"` and by using the dahsboards and telemetry data avaiable observe, debug and disagnose the problems.

| Feature Flag                      | Service          | Description                                                                                               |
| --------------------------------- | ---------------- | --------------------------------------------------------------------------------------------------------- |
| adServiceFailure                  | Ad Service       | Generate an error for GetAds 1/10th of the time                                                           |
| adServiceManualGc                 | Ad Service       | Trigger full manual garbage collections in the ad service                                                 |
| adServiceHighCpu                  | Ad Service       | Trigger high cpu load in the ad service. If you want to demo cpu throttling, set cpu resource limits      |
| cartServiceFailure                | Cart Service     | Generate an error for EmptyCart 1/10th of the time                                                        |
| productCatalogFailure             | Product Catalog  | Generate an error for GetProduct requests with product ID: OLJCESPC7Z                                     |
| recommendationServiceCacheFailure | Recommendation   | Create a memory leak due to an exponentially growing cache. 1.4x growth, 50% of requests trigger growth.  |
| paymentServiceFailure             | Payment Service  | Generate an error when calling the charge method.                                                         |
| paymentServiceUnreachable         | Checkout Service | Use a bad address when calling the PaymentService to make it seem like the PaymentService is unavailable. |
| loadgeneratorFloodHomepage        | Loadgenerator    | Start flooding the homepage with a huge amount of requests, configurable by changing flagd JSON on state. |
| kafkaQueueProblems                | Kafka            | Overloads Kafka queue while simultaneously introducing a consumer side delay leading to a lag spike.      |
| imageSlowLoad                     | Frontend         | Utilizes envoy fault injection, produces a delay in loading of product images in the frontend.            |

## Exercise: Diagnosing Intermittent Failures in AdService
In this exercise, you will diagnose intermittent failures in the AdService of our distributed system. You will use OpenTelemetry to collect telemetry data and use it to identify the root cause of the failures.

### Setup
To run this scenario, you will need to have our demo application running and the adServiceFailure feature flag enabled. 
This is enabled by opening the src/flagd/demo.flagd.json file and setting the defaultVariant to on like so:
```json
jsonCopy"adServiceFailure": {
"description": "Fail ad service",
"state": "ENABLED",
"variants": {
"on": true,
"off": false
},
"defaultVariant": "on"
}
```
Save the file and let the application run for a couple of minutes to allow for data to populate.

### Diagnosis
The first step in diagnosing a problem is to determine that a problem exists. Start by checking your metrics dashboard for any anomalies related to the AdService.
Open the Grafana dashboard and look for any unusual patterns in the AdService metrics, such as increased error rates or latency spikes.

### Grafana Tempo
Use Grafana Tempo to search for traces involving the AdService. Look for traces with errors or unusually high latency.
Confirming the Diagnosis
Inspect the trace details for traces that show failures in the AdService.

Question: Can you identify the pattern of failures in the AdService? How frequently do they occur?

<details>
<summary>Hint</summary>
Look at the code in AdService.java. There's a condition that throws an exception 1/10 of the time when the feature flag is enabled.
</details>

Question: What impact do these intermittent failures have on the overall system?

<details>
<summary>Hint</summary>
Consider how often ads are requested in the application and how a failure in the AdService might affect the user experience.
</details>

### Conclusion
In this scenario, we used OpenTelemetry to diagnose intermittent failures in our AdService. We used metrics to identify that there was a problem, and traces to confirm our suspicions and find the root cause of the issue.
Discuss how you would address this issue in a real-world scenario. What changes would you make to improve the reliability of the AdService?

## Exercise: Analyzing Manual Garbage Collection Impact on AdService
In this exercise, you will analyze the impact of manual garbage collection triggers on the AdService performance. You will use OpenTelemetry to collect telemetry data and use it to identify the effects of frequent garbage collection.
### Setup
To run this scenario, you will need to have our demo application running and the adServiceManualGc feature flag enabled. This is enabled by opening the src/flagd/demo.flagd.json file and setting the defaultVariant to on like so:
```json
jsonCopy"adServiceManualGc": {
"description": "Triggers full manual garbage collections in the ad service",
"state": "ENABLED",
"variants": {
"on": true,
"off": false
},
"defaultVariant": "on"
}
```
Save the file and let the application run for a couple of minutes to allow for data to populate.
### Diagnosis
Start by examining your metrics dashboard for any unusual patterns in the AdService performance metrics.
Open the Grafana dashboard and look for any anomalies in the AdService metrics, particularly focusing on response times, CPU usage, and memory patterns.
### Grafana Tempo
Use Grafana Tempo to search for traces involving the AdService. Look for traces with unusually high latency or patterns of periodic slowdowns.
### Confirming the Diagnosis
Inspect the trace details for traces that show slow performance in the AdService.

Question: Can you identify periods of increased latency in the AdService? How do they correlate with garbage collection events?

<details>
<summary>Hint</summary>
Look at the code in GarbageCollectionTrigger.java. It's triggering manual garbage collection at regular intervals.
</details>

Question: What impact does frequent manual garbage collection have on the AdService's performance and resource usage?

<details>
<summary>Hint</summary>
Consider how garbage collection affects CPU usage and response times, especially when it occurs frequently.
</details>

### Conclusion
In this scenario, we used OpenTelemetry to analyze the impact of manual garbage collection on our AdService. We used metrics to identify performance issues, and traces to confirm the correlation between garbage collection events and service slowdowns.
Discuss the pros and cons of manual garbage collection in a production environment. How would you optimize the garbage collection strategy for the AdService?

## Exercise: Investigating High CPU Usage in AdService
In this exercise, you will investigate high CPU usage in the AdService of our distributed system. You will use OpenTelemetry to collect telemetry data and use it to identify the cause and impact of the high CPU usage.
## Setup
To run this scenario, you will need to have our demo application running and the adServiceHighCpu feature flag enabled. This is enabled by opening the src/flagd/demo.flagd.json file and setting the defaultVariant to on like so:
```json
jsonCopy"adServiceHighCpu": {
"description": "Triggers high cpu load in the ad service",
"state": "ENABLED",
"variants": {
"on": true,
"off": false
},
"defaultVariant": "on"
}
```
Save the file and let the application run for a couple of minutes to allow for data to populate.

### Diagnosis
Begin by examining your metrics dashboard for any unusual patterns in the AdService performance metrics.
Open the Grafana dashboard and look for anomalies in the AdService metrics, particularly focusing on CPU usage, response times, and request throughput.
### Grafana Tempo
Use Grafana Tempo to search for traces involving the AdService. Look for traces with high execution times or patterns of consistently slow performance.
### Confirming the Diagnosis
Inspect the trace details for traces that show high CPU usage or slow performance in the AdService.

Question: Can you identify the cause of high CPU usage in the AdService? What specific operations are consuming the most CPU?

<details>
<summary>Hint</summary>
Look at the code in CPULoad.java. It's spawning threads that perform CPU-intensive calculations.
</details>

Question: How does the high CPU usage affect the overall performance and responsiveness of the AdService?

<details>
<summary>Hint</summary>
Consider how high CPU usage might impact response times, throughput, and the ability to handle concurrent requests.
</details>

### Conclusion
In this scenario, we used OpenTelemetry to investigate high CPU usage in our AdService. We used metrics to identify performance issues, and traces to pinpoint the specific operations causing high CPU consumption.
Discuss strategies for mitigating high CPU usage in a microservice environment. How would you redesign the AdService to be more CPU-efficient while still meeting its functional requirements?

## Exercise: Diagnosing Intermittent Cart Service Failures
Now it is time to put your skills to the test. In this exercise, you will diagnose intermittent failures in the cart service of a distributed system. You will use OpenTelemetry to collect telemetry data and use it to identify the root cause of the failures.
Application telemetry, such as the kind that OpenTelemetry can provide, is very useful for diagnosing issues in a distributed system. In this scenario, we will walk through a scenario demonstrating how to move from high-level metrics and traces to determine the cause of intermittent failures in the cart service.
Setup
To run this scenario, you will need to have our demo application running and the cartServiceFailure feature flag enabled. This is enabled by opening the src/flagd/demo.flagd.json file and setting the defaultVariant to on like so:

```json
jsonCopy"cartServiceFailure": {
"description": "Fail cart service",
"state": "ENABLED",
"variants": {
"on": true,
"off": false
},
"defaultVariant": "on"
}
```
Save the file and let the application run for a couple of minutes to allow for data to populate.
### Diagnosis
The first step in diagnosing a problem is to determine that a problem exists. Often the first stop will be a the metrics dashboard that we created in the previous exercises. I told you some of those metrics would come in handy one day, didn't I?!
Open the Grafana dashboard and look for any anomalies in the metrics. You should see that the cart service is experiencing intermittent failures.
Cart Service charts are generated from OpenTelemetry Metrics exported to Prometheus, while the Service Latency and Error Rate charts are generated through the Span Metrics processor.
From our dashboard, we can see that there seems to be anomalous behavior in the cart service – occasional spikes in error rates, as well as intermittent increases in latency in our p95, 99, and 99.9 histograms.
We know that we're emitting trace data from our application as well, so let's think about another way that we'd be able to determine that a problem exists.
### Grafana Tempo
Grafana Tempo allows us to search for traces and display the end-to-end latency of an entire request with visibility into each individual part of the overall request. Perhaps we noticed an increase in error rates on our cart operations. Tempo allows us to then search and filter our traces to include only those that include requests to the cart service.
By sorting by status code, we're able to quickly find specific traces that resulted in errors. Clicking on a trace we're able to view the waterfall view.
We can see that the cart service is occasionally failing to complete its work, and viewing the details allows us to get a better idea of what's going on.
Confirming the Diagnosis
Inspect the trace details for a trace that resulted in an error.
!!! note
Copy:question: Can you spot any anomalies in the trace details such as span attributes or tags that might indicate the root cause of the intermittent failures?

<details>
<summary>Hint</summary>

Look for attributes related to the cart operation, particularly for the `EmptyCart` operation. You might notice that errors occur only for a subset of requests.
</details>

:question: Can you find previous traces that exhibit the same or different behavior?

<details>
<summary>Hint</summary>

Try filtering for `cartservice` in the service name, and search for successful and failed `EmptyCart` operations. Compare the attributes and timings between successful and failed requests. You should notice that failures occur intermittently.
</details>

### Conclusion
Now, since this is a contrived scenario, we know where to find the underlying issue in our code. However, in a real-world scenario, we may need to perform further searching to find out what's going on in our code, or the interactions between services that cause it.
In this scenario, we were able to use OpenTelemetry to diagnose intermittent failures in our cart service. We were able to use metrics to identify that there was a problem, and traces to confirm our suspicions and find the root cause of the issue.

### Exercise: Diagnosing Product Catalog Service Failure
In this exercise, you will diagnose a failure in the product catalog service of our demo application. You will use OpenTelemetry to collect telemetry data and use it to identify the root cause of the issue.
## Setup
To run this scenario, you will need to have our demo application running and the productCatalogFailure feature flag enabled. This is enabled by opening the src/flagd/demo.flagd.json file and setting the defaultVariant to on like so:

```json
jsonCopy"productCatalogFailure": {
"description": "Fail product catalog service on a specific product",
"state": "ENABLED",
"variants": {
"on": true,
"off": false
},
"defaultVariant": "on"
}
```

Save the file and let the application run for a couple of minutes to allow for data to populate.
### Diagnosis
Start by examining the metrics dashboard in Grafana. Look for any anomalies in the metrics related to the product catalog service. 
You should see increased error rates and potentially higher latency for this service.
Next, use Grafana Tempo to search for traces that include the product catalog service. Sort by error status or high latency to find problematic traces quickly.
### Confirming the Diagnosis
Inspect the trace details for a trace that shows an error or took a long time to complete.
!!! note
Copy:question: Can you identify which specific product ID is causing the failure?

<details>
<summary>Hint</summary>

Look for the `app.product.id` attribute in the span details. Is there a particular ID that consistently appears in failing requests?
</details>

:question: What's the error message associated with the failing requests?

<details>
<summary>Hint</summary>

Check the `error.message` attribute in the failing spans. It should mention something about a feature flag being enabled.
</details>
Code Investigation
Now that we've identified the problematic product ID and error message, let's look at the relevant code in the main.go file of the product catalog service.
Find the checkProductFailure function. What does it do?
!!! note
Copy:question: How does the code determine whether to simulate a failure?

<details>
<summary>Hint</summary>

The function checks if the product ID is "OLJCESPC7Z" and if the `productCatalogFailure` feature flag is enabled.
</details>

### Conclusion
In this exercise, we used OpenTelemetry and observability tools to diagnose a simulated failure in the product catalog service. We identified that:

* The failure only occurs for a specific product ID.
* The failure is triggered by a feature flag.
* The error is intentionally introduced in the code to simulate a real-world scenario where a specific product might cause issues.

This demonstrates how feature flags can be used to test error handling and how observability tools can help quickly identify and diagnose issues in a microservices architecture.

## Exercise: Diagnosing a Cache Failure in the Recommendation Service
In this exercise, you will diagnose a cache failure in the recommendation service of our distributed system. You will use OpenTelemetry to collect telemetry data and use it to identify the root cause of the problem.
### Setup
To run this scenario, you will need to have our demo application running and the recommendationServiceCacheFailure feature flag enabled. This is enabled by opening the src/flagd/demo.flagd.json file and setting the defaultVariant to on like so:
```json
"recommendationServiceCacheFailure": {
"description": "Fail recommendation service cache",
"state": "ENABLED",
"variants": {
"on": true,
"off": false
},
"defaultVariant": "on"
}
```

Save the file and let the application run for a couple of minutes to allow for data to populate.
### Diagnosis

Start by examining the Grafana dashboard. Look for any anomalies in the metrics related to the recommendation service.
Pay special attention to:

* CPU utilization
* Memory usage
* Service latency (p95, p99, and p99.9 percentiles)
* Error rates


Once you've identified the anomalies, use Grafana Tempo to search for traces that include requests to the recommendation service.
Sort the traces by latency and examine the waterfall view of a long-running trace.

### Questions to Answer

What patterns do you notice in the CPU and memory usage of the recommendation service?
How does the latency of the recommendation service compare to its normal behavior?
In the trace details, what value does the app.cache_hit attribute have for slow requests?
What is unusual about the app.products.count value in the problematic traces?
Compare traces with app.cache_hit=true and app.cache_hit=false. What differences do you notice in terms of latency and behavior?

### Code Investigation
Now that you've identified the problem through observability data, it's time to look at the code. Examine the recommendation_server.py file, focusing on the get_product_list function.

How does the code behave differently when the recommendationServiceCacheFailure flag is enabled?
What happens to the cached_ids list over time when the cache "misses"?
Can you identify the potential memory leak in this code?

### Conclusion
Based on your analysis of the telemetry data and code review:

Explain the root cause of the cache failure and memory leak in the recommendation service.
Propose a fix for the issue.
Describe how you would validate that your fix resolves the problem using OpenTelemetry data.

Remember, in a real-world scenario, problems are often more complex and may require deeper investigation. This exercise demonstrates the power of OpenTelemetry in helping to quickly identify and diagnose issues in distributed systems.

