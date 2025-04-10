apiVersion: helm.toolkit.fluxcd.io/v2beta2
kind: HelmRelease
metadata:
  name: prometheus
  namespace: kube-system
spec:
  chart:
    spec:
      chart: prometheus
      version: "25.16.0"
      sourceRef:
        kind: HelmRepository
        name: addons
  # Default values
  # https://gitlab.verizon.com/go0v-containers/charts/prometheus/-/blob/main/values.yaml
  values:
    imagePullSecrets:
      - name: "go0v-vzdocker-config"
    configmapReload:
      prometheus:
        image:
          repository: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/prometheus-config-reloader
          tag: v0.71.2
        containerSecurityContext:
          allowPrivilegeEscalation: false
          capabilities: 
            drop: 
              - ALL
          privileged: false
          readOnlyRootFilesystem: true
          runAsGroup: 65534
          runAsNonRoot: true
          runAsUser: 65534
    server:
      image:
        repository: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/prometheus
        tag: v2.50.1
      containerSecurityContext:
        allowPrivilegeEscalation: false
        capabilities: 
          drop: 
            - ALL
        privileged: false
        readOnlyRootFilesystem: true
        runAsGroup: 65534
        runAsNonRoot: true
        runAsUser: 65534
      service:
        type: NodePort
      resources:
        limits:
          cpu: 60m
          memory: 1Gi
        requests:
          cpu: 40m
          memory: 800Mi
      verticalAutoscaler:
        enabled: true
      podDisruptionBudget:
        enabled: true
        minAvailable: 0
    alertmanager:
      image:
        repository:  go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/alertmanager
        tag: v0.27.0
      imagePullSecrets:
      - name: "go0v-vzdocker-config"
      securityContext:
        allowPrivilegeEscalation: false
        capabilities: 
          drop: 
            - ALL
        privileged: false
        readOnlyRootFilesystem: true
        runAsGroup: 65534
        runAsNonRoot: true
        runAsUser: 65534
      resources:
        limits:
          cpu: 3m
          memory: 60Mi
        requests:
          cpu: 1m
          memory: 26Mi
    kube-state-metrics:
      enabled: false
    prometheus-node-exporter:
      image:
        registry: go0v-vzdocker.oneartifactoryprod.verizon.com
        repository: containers/cicd/prometheus-node-exporter
        tag: v1.7.0
      imagePullSecrets:
      - name: "go0v-vzdocker-config"
      containerSecurityContext:
        allowPrivilegeEscalation: false
        capabilities: 
          drop: 
            - ALL
        privileged: false
        readOnlyRootFilesystem: true
        runAsGroup: 65534
        runAsNonRoot: true
        runAsUser: 65534
      resources:
        limits:
          cpu: 6m
          memory: 50Mi
        requests:
          cpu: 4m
          memory: 24Mi
    prometheus-pushgateway:
      enabled: false
    serverFiles:
      prometheus.yml:
        scrape_configs:
          - job_name: prometheus
            static_configs:
              - targets:
                - localhost:9090
          - job_name: 'kubernetes-apiservers'
            kubernetes_sd_configs:
              - role: endpoints
            scheme: https
            tls_config:
              ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
              insecure_skip_verify: true
            bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
            relabel_configs:
              - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
                action: keep
                regex: default;kubernetes;https
          - job_name: 'kubernetes-event-exporter'
            kubernetes_sd_configs:
              - role: endpoints
            relabel_configs:
            - source_labels: [__meta_kubernetes_endpoints_name]
              regex: 'prometheus-k8s-events-exporter'
              action: keep
          - job_name: 'kubernetes-node-exporter'
            kubernetes_sd_configs:
              - role: endpoints
            relabel_configs:
            - source_labels: [__meta_kubernetes_endpoints_name]
              regex: 'prometheus-prometheus-node-exporter'
              action: keep
          - job_name: 'kubernetes-nodes'
            scheme: https
            tls_config:
              ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
              insecure_skip_verify: true
            bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
            kubernetes_sd_configs:
              - role: node
            relabel_configs:
              - action: labelmap
                regex: __meta_kubernetes_node_label_(.+)
              - target_label: __address__
                replacement: kubernetes.default.svc:443
              - source_labels: [__meta_kubernetes_node_name]
                regex: (.+)
                target_label: __metrics_path__
                replacement: /api/v1/nodes/$1/proxy/metrics
          - job_name: 'kubernetes-nodes-cadvisor'
            scheme: https
            tls_config:
              ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
              insecure_skip_verify: true
            bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
            kubernetes_sd_configs:
              - role: node
            relabel_configs:
              - action: labelmap
                regex: __meta_kubernetes_node_label_(.+)
              - target_label: __address__
                replacement: kubernetes.default.svc:443
              - source_labels: [__meta_kubernetes_node_name]
                regex: (.+)
                target_label: __metrics_path__
                replacement: /api/v1/nodes/$1/proxy/metrics/cadvisor
          - job_name: 'kubernetes-service-endpoints'
            honor_labels: true
            kubernetes_sd_configs:
              - role: endpoints
            relabel_configs:
              - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
                action: keep
                regex: true
              - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape_slow]
                action: drop
                regex: true
              - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
                action: replace
                target_label: __scheme__
                regex: (https?)
              - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
                action: replace
                target_label: __metrics_path__
                regex: (.+)
              - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
                action: replace
                target_label: __address__
                regex: (.+?)(?::\d+)?;(\d+)
                replacement: $1:$2
              - action: labelmap
                regex: __meta_kubernetes_service_annotation_prometheus_io_param_(.+)
                replacement: __param_$1
              - action: labelmap
                regex: __meta_kubernetes_service_label_(.+)
              - source_labels: [__meta_kubernetes_namespace]
                action: replace
                target_label: namespace
              - source_labels: [__meta_kubernetes_service_name]
                action: replace
                target_label: service
              - source_labels: [__meta_kubernetes_pod_node_name]
                action: replace
                target_label: node
              - action: drop
                regex: prometheus-node-exporter
                source_labels:
                - __meta_kubernetes_endpoints_label_app_kubernetes_io_name
          - job_name: 'kubernetes-service-endpoints-slow'
            honor_labels: true
            scrape_interval: 5m
            scrape_timeout: 30s
            kubernetes_sd_configs:
              - role: endpoints
            relabel_configs:
              - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape_slow]
                action: keep
                regex: true
              - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
                action: replace
                target_label: __scheme__
                regex: (https?)
              - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
                action: replace
                target_label: __metrics_path__
                regex: (.+)
              - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
                action: replace
                target_label: __address__
                regex: (.+?)(?::\d+)?;(\d+)
                replacement: $1:$2
              - action: labelmap
                regex: __meta_kubernetes_service_annotation_prometheus_io_param_(.+)
                replacement: __param_$1
              - action: labelmap
                regex: __meta_kubernetes_service_label_(.+)
              - source_labels: [__meta_kubernetes_namespace]
                action: replace
                target_label: namespace
              - source_labels: [__meta_kubernetes_service_name]
                action: replace
                target_label: service
              - source_labels: [__meta_kubernetes_pod_node_name]
                action: replace
                target_label: node
          - job_name: 'kubernetes-services'
            honor_labels: true
            metrics_path: /probe
            params:
              module: [http_2xx]
            kubernetes_sd_configs:
              - role: service
            relabel_configs:
              - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
                action: keep
                regex: true
              - source_labels: [__address__]
                target_label: __param_target
              - target_label: __address__
                replacement: blackbox
              - source_labels: [__param_target]
                target_label: instance
              - action: labelmap
                regex: __meta_kubernetes_service_label_(.+)
              - source_labels: [__meta_kubernetes_namespace]
                target_label: namespace
              - source_labels: [__meta_kubernetes_service_name]
                target_label: service
          - job_name: 'kubernetes-pods'
            honor_labels: true
            kubernetes_sd_configs:
              - role: pod
            relabel_configs:
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
                action: keep
                regex: true
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape_slow]
                action: drop
                regex: true
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scheme]
                action: replace
                regex: (https?)
                target_label: __scheme__
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
                action: replace
                target_label: __metrics_path__
                regex: (.+)
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port, __meta_kubernetes_pod_ip]
                action: replace
                regex: (\d+);(([A-Fa-f0-9]{1,4}::?){1,7}[A-Fa-f0-9]{1,4})
                replacement: '[$2]:$1'
                target_label: __address__
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port, __meta_kubernetes_pod_ip]
                action: replace
                regex: (\d+);((([0-9]+?)(\.|$)){4})
                replacement: $2:$1
                target_label: __address__
              - action: labelmap
                regex: __meta_kubernetes_pod_annotation_prometheus_io_param_(.+)
                replacement: __param_$1
              - action: labelmap
                regex: __meta_kubernetes_pod_label_(.+)
              - source_labels: [__meta_kubernetes_namespace]
                action: replace
                target_label: namespace
              - source_labels: [__meta_kubernetes_pod_name]
                action: replace
                target_label: pod
              - source_labels: [__meta_kubernetes_pod_phase]
                regex: Pending|Succeeded|Failed|Completed
                action: drop
              - source_labels: [__meta_kubernetes_pod_node_name]
                action: replace
                target_label: node
          - job_name: 'kubernetes-pods-slow'
            honor_labels: true
            scrape_interval: 5m
            scrape_timeout: 30s
            kubernetes_sd_configs:
              - role: pod
            relabel_configs:
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape_slow]
                action: keep
                regex: true
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scheme]
                action: replace
                regex: (https?)
                target_label: __scheme__
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
                action: replace
                target_label: __metrics_path__
                regex: (.+)
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port, __meta_kubernetes_pod_ip]
                action: replace
                regex: (\d+);(([A-Fa-f0-9]{1,4}::?){1,7}[A-Fa-f0-9]{1,4})
                replacement: '[$2]:$1'
                target_label: __address__
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port, __meta_kubernetes_pod_ip]
                action: replace
                regex: (\d+);((([0-9]+?)(\.|$)){4})
                replacement: $2:$1
                target_label: __address__
              - action: labelmap
                regex: __meta_kubernetes_pod_annotation_prometheus_io_param_(.+)
                replacement: __param_$1
              - action: labelmap
                regex: __meta_kubernetes_pod_label_(.+)
              - source_labels: [__meta_kubernetes_namespace]
                action: replace
                target_label: namespace
              - source_labels: [__meta_kubernetes_pod_name]
                action: replace
                target_label: pod
              - source_labels: [__meta_kubernetes_pod_phase]
                regex: Pending|Succeeded|Failed|Completed
                action: drop
              - source_labels: [__meta_kubernetes_pod_node_name]
                action: replace
                target_label: node
