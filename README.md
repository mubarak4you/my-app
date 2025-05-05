🧮 Total CPU Request of Higher-Priority DaemonSets (excluding Sysdig):

💡 Total = 3,480m = 3.48 cores
📌 Count:

    Total DaemonSets with higher priority: 36

    Total with lower or no priority: 5
    
    
    
    ✅ DaemonSets with Higher Priority
NAMESPACE	NAME	CPU REQUEST
gke-managed-system	dcgm-exporter	100m
kube-system	anetd	205m
kube-system	anetd-l	205m
kube-system	anetd-m	205m
kube-system	anetd-s	205m
kube-system	anetd-win	2m
kube-system	anetd-xl	205m
kube-system	anetd-xs	205m
kube-system	filestore-node	25m
kube-system	fluentbit-gke	105m
kube-system	fluentbit-gke-256pd	855m
kube-system	fluentbit-gke-max	7m
kube-system	gke-metadata-server	100m
kube-system	gke-metrics-agent	19m
kube-system	gke-metrics-agent-scaling-10	19m
kube-system	gke-metrics-agent-scaling-100	19m
kube-system	gke-metrics-agent-scaling-20	19m
kube-system	gke-metrics-agent-scaling-200	19m
kube-system	gke-metrics-agent-scaling-50	19m
kube-system	gke-metrics-agent-scaling-500	19m
kube-system	gke-metrics-agent-windows	30m
kube-system	ip-masq-agent	10m
kube-system	istio-cni-node	100m
kube-system	kube-proxy	100m
kube-system	metadata-proxy-v0.1	32m
kube-system	netd	8m
kube-system	nvidia-driver-installer	150m
kube-system	nvidia-gpu-device-plugin-large-cos	55m
kube-system	nvidia-gpu-device-plugin-large-ubuntu	55m
kube-system	nvidia-gpu-device-plugin-medium-cos	55m
kube-system	nvidia-gpu-device-plugin-medium-ubuntu	55m
kube-system	nvidia-gpu-device-plugin-small-cos	55m
kube-system	nvidia-gpu-device-plugin-small-ubuntu	55m
kube-system	pdcsi-node	10m
kube-system	pdcsi-node-windows	10m
kube-system	runsc-metric-server	3m
kube-system	tpu-device-plugin	110m


These 5 DaemonSets are running at lower priority than Sysdig:

    gmp-system/collector

    kube-system/container-watcher

    kube-system/maintenance-handler

    kube-system/nccl-fastsocket-installer

    kube-system/network-metering-agent
    
    
    
    Match the highest priority currently in the cluster
    priorityClassName: system-node-critical
    set its priorityClass to equal or higher than all others competing for node resources.
    
    
        Ensure Sysdig is treated equally in preemption and scheduling with critical system DaemonSets like anetd, fluentbit, and gke-metadata-server.

    Prevent it from being preempted or blocked on smaller nodes during high CPU demand.

    ⚠️ Important: This does not guarantee it always schedules — but it prevents priority-based starvation. You still need to ensure there's enough available CPU across nodes.
    
    
    
    