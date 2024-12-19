A comprehensive stress test was conducted to evaluate the performance and resource utilization of the OCI ingress controller pods. During this test, 50 ingress classes were created simultaneously within a one-minute window. The results revealed noticeable spikes in both memory usage and CPU utilization, as observed in the respective monitoring graphs for Memory Usage and CPU Usage.

Based on the analysis of these results, the following resource allocation settings are recommended for optimal performance:

These configurations aim to ensure efficient resource utilization while maintaining the stability and performance of the OCI ingress controller pods under high-load conditions.





Horizontal Pod Autoscaler (HPA) and Vertical Pod Autoscaler (VPA) Analysis

Following an in-depth evaluation and testing, it was observed that requests directed to the OCI ingress controller pods were being processed by only one of the two available native controller pods. This behavior indicates that the system is not configured as an active-active setup. Consequently, the Horizontal Pod Autoscaler (HPA) is not required, as there is no need for dynamic scaling based on load.

Similarly, the Vertical Pod Autoscaler (VPA) does not need to be configured. The baseline analysis of CPU and memory utilization shows that the allocated resources are sufficient to handle workloads, even under stress test conditions. The ingress controller pod demonstrated the ability to manage requests efficiently without exceeding its resource limits.

Additionally, the default Helm chart configuration does not include scaling for multiple replicas. However, it does provide for failover redundancy. In the event one pod becomes unavailable due to resource exhaustion, the system is configured with two replicas, ensuring that the second pod seamlessly takes over operations.


