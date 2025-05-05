Step 1: Baseline Resource Audit

    Identify all DaemonSets with equal or higher priorityClass than the Sysdig agent.

    Calculate the total CPU request from these DaemonSets to assess baseline CPU request on new nodes.
    
    
    
    
gke-managed-system  dcgm-exporter                           100m
gmp-system          collector                               5m
kube-system         anetd                                   205m
kube-system         anetd-l                                 205m
kube-system         anetd-m                                 205m
kube-system         anetd-s                                 205m
kube-system         anetd-win                               2m
kube-system         anetd-xl                                205m
kube-system         anetd-xs                                205m
kube-system         container-watcher                       25m
kube-system         filestore-node                          25m
kube-system         fluentbit-gke                           105m
kube-system         fluentbit-gke-256pd                     855m
kube-system         fluentbit-gke-max                       7m
kube-system         gke-metadata-server                     100m
kube-system         gke-metrics-agent                       19m
kube-system         gke-metrics-agent-scaling-10            19m
kube-system         gke-metrics-agent-scaling-100           19m
kube-system         gke-metrics-agent-scaling-20            19m
kube-system         gke-metrics-agent-scaling-200           19m
kube-system         gke-metrics-agent-scaling-50            19m
kube-system         gke-metrics-agent-scaling-500           19m
kube-system         gke-metrics-agent-windows               30m
kube-system         ip-masq-agent                           10m
kube-system         istio-cni-node                          100m
kube-system         kube-proxy                              100m
kube-system         maintenance-handler                     0m
kube-system         metadata-proxy-v0.1                     32m
kube-system         nccl-fastsocket-installer               0m
kube-system         netd                                    8m
kube-system         network-metering-agent                  0m
kube-system         nvidia-driver-installer                 150m
kube-system         nvidia-gpu-device-plugin-large-cos      55m
kube-system         nvidia-gpu-device-plugin-large-ubuntu   55m
kube-system         nvidia-gpu-device-plugin-medium-cos     55m
kube-system         nvidia-gpu-device-plugin-medium-ubuntu  55m
kube-system         nvidia-gpu-device-plugin-small-cos      55m
kube-system         nvidia-gpu-device-plugin-small-ubuntu   55m
kube-system         pdcsi-node                              10m
kube-system         pdcsi-node-windows                      10m
kube-system         runsc-metric-server                     3m
kube-system         sysdig-agent                            250m
kube-system         tpu-device-plugin                       110m


The total CPU request for all DaemonSets excluding sysdig-agent is:

    3480 millicores
    3.48 CPU cores

Total vCPU on newly created clusters = 10
Total of 3 Nodes



[alabimu@10-74-129-4 ~]$ kubectl get daemonsets --all-namespaces -o custom-columns='NAMESPACE:.metadata.namespace,NAME:.metadata.name,PRIORITY:.spec.template.spec.priorityClassName'

NAMESPACE            NAME                                     PRIORITY
gke-managed-system   dcgm-exporter                            system-node-critical
gmp-system           collector                                gmp-critical
kube-system          anetd                                    system-node-critical
kube-system          anetd-l                                  system-node-critical
kube-system          anetd-m                                  system-node-critical
kube-system          anetd-s                                  system-node-critical
kube-system          anetd-win                                system-node-critical
kube-system          anetd-xl                                 system-node-critical
kube-system          anetd-xs                                 system-node-critical
kube-system          container-watcher                        <none>
kube-system          filestore-node                           system-node-critical
kube-system          fluentbit-gke                            system-node-critical
kube-system          fluentbit-gke-256pd                      system-node-critical
kube-system          fluentbit-gke-max                        system-node-critical
kube-system          gke-metadata-server                      system-node-critical
kube-system          gke-metrics-agent                        system-node-critical
kube-system          gke-metrics-agent-scaling-10             system-node-critical
kube-system          gke-metrics-agent-scaling-100            system-node-critical
kube-system          gke-metrics-agent-scaling-20             system-node-critical
kube-system          gke-metrics-agent-scaling-200            system-node-critical
kube-system          gke-metrics-agent-scaling-50             system-node-critical
kube-system          gke-metrics-agent-scaling-500            system-node-critical
kube-system          gke-metrics-agent-windows                system-node-critical
kube-system          ip-masq-agent                            system-node-critical
kube-system          istio-cni-node                           system-node-critical
kube-system          kube-proxy                               system-node-critical
kube-system          maintenance-handler                      <none>
kube-system          metadata-proxy-v0.1                      system-node-critical
kube-system          nccl-fastsocket-installer                <none>
kube-system          netd                                     system-node-critical
kube-system          network-metering-agent                   <none>
kube-system          nvidia-driver-installer                  system-node-critical
kube-system          nvidia-gpu-device-plugin-large-cos       system-node-critical
kube-system          nvidia-gpu-device-plugin-large-ubuntu    system-node-critical
kube-system          nvidia-gpu-device-plugin-medium-cos      system-node-critical
kube-system          nvidia-gpu-device-plugin-medium-ubuntu   system-node-critical
kube-system          nvidia-gpu-device-plugin-small-cos       system-node-critical
kube-system          nvidia-gpu-device-plugin-small-ubuntu    system-node-critical
kube-system          pdcsi-node                               system-node-critical
kube-system          pdcsi-node-windows                       system-node-critical
kube-system          runsc-metric-server                      system-node-critical
kube-system          sysdig-agent                             system-cluster-critical
kube-system          tpu-device-plugin                        system-node-critical

[alabimu@10-74-129-4 ~]$ kubectl get priorityClass
NAME                      VALUE        GLOBAL-DEFAULT   AGE
gmp-critical              1000000000   false            171m
overprovisioning          -10          false            165m
system-cluster-critical   2000000000   false            174m
system-node-critical      2000001000   false            174m