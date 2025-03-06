deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "istio-ingressgateway.fullname" . }}
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "istio-ingressgateway.labels" . | nindent 4 }}
  {{- with .Values.deploymentAnnotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  selector:
    matchLabels:
      {{- include "istio-ingressgateway.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        # This is required to tell Anthos Service Mesh to inject the gateway with the
        # required configuration.
        inject.istio.io/templates: gateway
      labels:
        {{- include "istio-ingressgateway.selectorLabels" . | nindent 8 }}
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
        - name: {{ .Values.imagePullSecret }}
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
      serviceAccountName: {{ include "istio-ingressgateway.serviceAccountName" . }}




_helpers.tpl
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


values.yaml
resources:
  limits:
    cpu: 2000m
    memory: 1024Mi
  requests:
    cpu: 100m
    memory: 128Mi

serviceAccount:
  create: true
  annotations: {}
  name: "istio-ingressgateway-test"
  secrets: []

imagePullSecret: go0v-vzdocker

deploymentAnnotations: {}


rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: {{ include "istio-ingressgateway.fullname" . }}
  namespace: {{ .Release.Namespace }}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: {{ include "istio-ingressgateway.fullname" . }}
subjects:
- kind: ServiceAccount
  name: {{ include "istio-ingressgateway.serviceAccountName" . }}

but service account that gets created the name is wrong..
[alabimu@10-74-128-76 ~]$ kubectl describe rolebinding -n asm-ingressgateway
Name:         istio-ingressgateway
Labels:       app.kubernetes.io/managed-by=configmanagement.gke.io
              applyset.kubernetes.io/part-of=applyset-NofvAQylE1-JR-RUcEJO4beYGo7H6CFrRpucNW5fWjM-v1
              configsync.gke.io/declared-version=v1
Annotations:  config.k8s.io/owning-inventory: config-management-system_istio-ingressgateway
              configmanagement.gke.io/cluster-name: gke-etgke-1237-np-us-east4-go0v
              configmanagement.gke.io/managed: enabled
              configmanagement.gke.io/source-path: istio-ingressgateway/templates/rolebinding.yaml
              configmanagement.gke.io/token: 1.0.1
              configsync.gke.io/declared-fields:
                {"f:metadata":{"f:annotations":{"f:configmanagement.gke.io/cluster-name":{},"f:configmanagement.gke.io/source-path":{}},"f:labels":{"f:con...
              configsync.gke.io/git-context: {"repo":"oci://go0v-vzdocker.oneartifactoryprod.verizon.com/containers/dev/charts","rev":"1.0.1"}
              configsync.gke.io/manager: :root_istio-ingressgateway
              configsync.gke.io/resource-id: rbac.authorization.k8s.io_rolebinding_asm-ingressgateway_istio-ingressgateway
Role:
  Kind:  Role
  Name:  istio-ingressgateway
Subjects:
  Kind            Name                  Namespace
  ----            ----                  ---------
  ServiceAccount  istio-ingressgateway  



