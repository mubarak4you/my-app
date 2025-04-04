apiVersion: v1
kind: Pod
metadata:
  name: prometheus-components
spec:
  containers:
    - name: prometheus-alertmanager
      image: prom/alertmanager
      resources:
        requests:
          cpu: "1m"
          memory: "28Mi"
        limits:
          cpu: "3m"
          memory: "60Mi"

    - name: prometheus-node-exporter
      image: prom/node-exporter
      resources:
        requests:
          cpu: "2m"
          memory: "24Mi"
        limits:
          cpu: "4m"
          memory: "50Mi"

    - name: prometheus-server
      image: prom/prometheus
      resources:
        requests:
          cpu: "8m"
          memory: "540Mi"
        limits:
          cpu: "20m"
          memory: "1Gi"



    prometheus-alertmanager

        CPU (observed): 0.796 mcores

        Memory (observed): 23.5 MiB

        CPU Request: 1m

        CPU Limit: 3m

        Memory Request: 28Mi

        Memory Limit: 60Mi

    prometheus-node-exporter

        CPU (observed): 1.23 mcores

        Memory (observed): 19.3 MiB

        CPU Request: 2m

        CPU Limit: 4m

        Memory Request: 24Mi

        Memory Limit: 50Mi

    prometheus-server

        CPU (observed): 6.90 mcores

        Memory (observed): 450 MiB

        CPU Request: 8m

        CPU Limit: 20m

        Memory Request: 540Mi

        Memory Limit: 1Gi