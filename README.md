[alabimu@10-74-129-4 ~]$ kubectl get daemonsets -A -o json | jq -r '
>   .items[] | 
>   {
>     ns: .metadata.namespace,
>     name: .metadata.name,
>     total_cpu: (
>       [.spec.template.spec.containers[].resources.requests.cpu // "0m"]
>       | map(
>           sub("m$"; "") 
>           | if test("^[0-9]+$") then tonumber else 0 end
>         )
>       | add
>     )
>   } 
>   | "\(.ns) \(.name) \(.total_cpu)m"
> ' | column -t
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