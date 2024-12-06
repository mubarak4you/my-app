Optimization of Cost and Performance: Grafana Resource Analysis
Memory Usage

The baseline memory usage of the Grafana pod, as observed over the last hour, is stable at approximately 74 MiB. However, memory usage spikes when the "Total Pod Memory Usage" and "Pod CPU Usage (mcores)" dashboards are opened in the browser.
Stress Testing

    Test 1:
        Sequential Requests:
            Sent 25 HTTP requests one after the other using a curl script that authenticates with Grafana admin credentials.
            Each request triggers a Prometheus scrape to retrieve metrics.
        Concurrent Requests:
            Sent 25 overlapping HTTP requests to the dashboard, simulating multiple users accessing the dashboard simultaneously.
            The sequential request target was met five times.

    Test 2:
        Similar tests were performed with Sequential and Concurrent Requests, but with only 5 requests each.

    Test 3:
        Opened 5 dashboard pages manually in the browser and refreshed each page to trigger loading and Prometheus metric scraping.

Results:

    Memory usage spiked but remained below 108 MiB across all tests.
    Recommended settings:
        Memory Request: 85 MiB
        Memory Limit: 130 MiB

Configuration:

resources:
  requests:
    memory: "85Mi"
  limits:
    memory: "130Mi"

CPU Usage

The baseline CPU usage of the Grafana pod is stable at approximately 1.5 mcores but increases significantly when dashboards are accessed in the browser.

Results:

    CPU usage spiked but did not exceed 5.5 mcores during testing.
    Recommended settings:
        CPU Request: 3 mcores
        CPU Limit: 7 mcores

Configuration:

resources:
  requests:
    cpu: "3m"
  limits:
    cpu: "7m"

Combined Memory and CPU Recommendations

Initial Resource Settings:

resources:
  limits:
    cpu: "20m"
    memory: "130Mi"
  requests:
    cpu: "8m"
    memory: "80Mi"

Observation:
After applying these values in the Grafana configuration and creating a new cluster, the Grafana web interface and dashboards exhibited slower load times.

Adjusted Resource Settings:

resources:
  limits:
    cpu: "60m"
    memory: "180Mi"
  requests:
    cpu: "40m"
    memory: "100Mi"
persistence:
