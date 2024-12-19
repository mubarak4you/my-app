A comprehensive stress test was conducted to evaluate the performance and resource utilization of the OCI ingress controller pods. During this test, 50 ingress classes were created simultaneously within a one-minute window. The results revealed noticeable spikes in both memory usage and CPU utilization, as observed in the respective monitoring graphs for Memory Usage and CPU Usage.

Based on the analysis of these results, the following resource allocation settings are recommended for optimal performance:

Resource Configuration:

    Limits:
        CPU: 12m
        Memory: 128Mi
    Requests:
        CPU: 2m
        Memory: 50Mi

These configurations aim to ensure efficient resource utilization while maintaining the stability and performance of the OCI ingress controller pods under high-load conditions.