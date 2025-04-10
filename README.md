etoke-311
Optimize cost and performance - prometheus

Acceptance Criteria
Configure the following for each workload.

1. Configure Resource Requests and Limits

Requests: Define the minimum amount of CPU and memory resources a container requires to start and run. Setting appropriate requests helps the scheduler place pods on suitable nodes.
Limits: Set the maximum amount of CPU and memory a container can consume. This prevents resource starvation for other workloads.
2. Pod Autoscaling (HPA/VPA):
Apply an autoscaling technique that is suited for the application:

Horizontal Pod Autoscaler (HPA): Automatically scales the number of pod replicas based on observed CPU utilization, memory usage, or custom metrics.
Vertical Pod Autoscaler (VPA): Adjusts the resource requests and limits of pods dynamically based on historical usage patterns.
3. Pod Disruption Budget (PDB)

Specifies the maximum number or percentage of pod replicas that can be unavailable simultaneously during voluntary disruptions, such as upgrades or node draining.




etoke-311

Prometheus   Baseline Values. (Last Hour) 
prometheus-alertmanager
CPU
0.796 mcores

Mem
23.5 MIB

prometheus-node-exporter
CPU
1.23 mcores

Mem
19.3 MIB

prometheus-server
CPU
6.90 mcores

Mem
450 MiB




prometheus-alertmanager

        CPU (observed): 0.796 mcores

        Memory (observed): 23.5 MiB
  Conducted a stress test using curl to send a burst of alerts (300 alerts) in the span of 1 minute using a loop.  
Memory after stress test.
￼
  CPU after stress test
￼
 Based on stress test will go with the following. 

resources:
  limits:
    cpu: “3m"
    memory: “50Mi"
  requests:
    cpu: “1m"
    memory: “24Mi"   

prometheus-node-exporter

        CPU (observed): 1.23 mcores

        Memory (observed): 19.3 MiB


Prometheus-node-exporter works passively to collect metrics that simply exposes node-level metrics (CPU, memory), its resource usage remains very low and consistent under normal conditions. Stressing the pod itself doesn’t reflect actual workloads, as it doesn't perform any significant computation or memory-intensive tasks but based on the baseline values I collected, here are the recommended CPU and memory request/limit values below for prometheus-node-exporter. These values provide a safe buffer over observed usage (1.23 mCPU and 19.3 MiB memory) while keeping the pod lightweight and scheduler-friendly.

resources:
  limits:
    cpu: “4m"
    memory: “50Mi"
  requests:
    cpu: “2m"
    memory: “24Mi"






prometheus-server

        CPU (observed): 6.90 mcores

        Memory (observed): 450 MiB


resources:
  limits:
    cpu: “30m"
    memory: “1Gi”
  requests:
    cpu: “15m”
    memory: “800Mi"


Performed a stress test on the prometheus-server that fires 5000 concurrent queries to using a heavy query: rate(container_cpu_usage_seconds_total[5m]), this loads both the PCU and memory by retrieving lots of time series data from TSDB 