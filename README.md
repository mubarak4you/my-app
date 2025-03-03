{{/*
Expand the name of the chart.
*/}}
{{- define "istio-ingressgateway.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
If release name contains chart name it will be used as a full name.
*/}}
{{- define "istio-ingressgateway.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "istio-ingressgateway.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "istio-ingressgateway.labels" -}}
helm.sh/chart: {{ include "istio-ingressgateway.chart" . }}
{{ include "istio-ingressgateway.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- if .Values.commonLabels }}
{{ toYaml .Values.commonLabels }}
{{- end }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "istio-ingressgateway.selectorLabels" -}}
app.kubernetes.io/name: {{ include "istio-ingressgateway.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Create the name of the service account to use
*/}}
{{- define "istio-ingressgateway.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "istio-ingressgateway.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}

{{/*
The image to use
*/}}
{{- define "istio-ingressgateway.image" -}}
{{- printf "%s:%s" .Values.image.repository (default (printf "v%s" .Chart.AppVersion) .Values.image.tag) }}
{{- end }}

{{/*
The image to use for the addon resizer
*/}}
{{- define "istio-ingressgateway.addonResizer.image" -}}
{{- printf "%s:%s" .Values.addonResizer.image.repository .Values.addonResizer.image.tag }}
{{- end }}

{{/*
ConfigMap name of addon resizer
*/}}
{{- define "istio-ingressgateway.addonResizer.configMap" -}}
{{- printf "%s-%s" (include "istio-ingressgateway.fullname" .) "nanny-config" }}
{{- end }}

{{/*
Role name of addon resizer
*/}}
{{- define "istio-ingressgateway.addonResizer.role" -}}
{{ printf "system:%s-nanny" (include "istio-ingressgateway.fullname" .) }}
{{- end }}

{{/* Get PodDisruptionBudget API Version */}}
{{- define "istio-ingressgateway.pdb.apiVersion" -}}
  {{- if and (.Capabilities.APIVersions.Has "policy/v1") (semverCompare ">= 1.21-0" .Capabilities.KubeVersion.Version) -}}
      {{- print "policy/v1" -}}
  {{- else -}}
    {{- print "policy/v1beta1" -}}
  {{- end -}}
{{- end -}}




apiVersion: apps/v1
kind: Deployment
metadata:
  name: istio-ingressgateway
spec:
  selector:
    matchLabels:
      app: istio-ingressgateway
      istio: ingressgateway
  template:
    metadata:
      annotations:
        # This is required to tell Anthos Service Mesh to inject the gateway with the
        # required configuration.
        inject.istio.io/templates: gateway
      labels:
        app: istio-ingressgateway
        istio: ingressgateway
        istio.io/rev: asm-managed-stable
    spec:
       # Allow binding to all ports (such as 80 and 443)
      securityContext:
        sysctls:
        - name: net.ipv4.ip_unprivileged_port_start
          value: "0"
      containers:
      - name: istio-proxy
        image: auto # The image will automatically update each time the pod starts.
        resources: {{ toYaml (.Values.resources) | nindent 10 }}
        securityContext: # https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          privileged: false
          readOnlyRootFilesystem: true
          runAsGroup: 1337
          runAsNonRoot: true
          runAsUser: 1337
      imagePullSecrets:
        - name: go0v-vzdocker
      # The istio proxy image will not resolve from "auto" until
      # ASM is provisioned.
      # This init container will delete it's pod until the image value
      # has resolved and ASM is ready
      initContainers:
      - image: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/kubectl:1.30.5
        name: init
        command: ["/bin/bash"]
        args:
          - "-c"
          - >
            export PROXY_IMAGE=$(kubectl -n $POD_NAMESPACE
            get pod $POD_NAME -o=jsonpath='{.spec.containers[0].image}');
            if [[ $PROXY_IMAGE == "auto" ]]; then
            sleep 30;
            kubectl -n $POD_NAMESPACE delete pod $POD_NAME;
            else
            echo "ASM is ready.";
            fi;
        securityContext:
          privileged: false
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          capabilities:
            drop: ["ALL"]
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
      serviceAccountName: {{ .Values.serviceAccountName }}




apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: istio-ingressgateway
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: istio-ingressgateway
  minReplicas: 3
  maxReplicas: 8
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80





apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: asm-ingressgateway
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: asm-ingressgateway
      istio: ingressgateway



apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: istio-ingressgateway
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "delete"]




apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: istio-ingressgateway
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: istio-ingressgateway
subjects:
- kind: ServiceAccount
  name: istio-ingressgateway


apiVersion: v1
kind: Service
metadata:
  name: istio-ingressgateway
  labels:
    app: istio-ingressgateway
    istio: ingressgateway
  annotations:
    networking.gke.io/load-balancer-type: "internal"
spec:
  ports:
  # status-port exposes a /healthz/ready endpoint that can be used with GKE Ingress health checks
  - name: status-port
    port: 15021
    protocol: TCP
    targetPort: 15021
  # Any ports exposed in Gateway resources should be exposed here.
  - name: https
    port: 443
  selector:
    istio: ingressgateway
    app: istio-ingressgateway
  type: LoadBalancer



apiVersion: v1
kind: ServiceAccount
metadata:
  name: istio-ingressgateway




values.yaml
resources:
  limits:
    cpu: 2000m
    memory: 1024Mi
  requests:
    cpu: 100m
    memory: 128Mi
serviceAccountName: istio-ingressgateway





