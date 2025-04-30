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
