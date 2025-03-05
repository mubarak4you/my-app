apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "istio-ingressgateway.fullname" . }}
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "istio-ingressgateway.labels" . | nindent 4 }}
  {{- with .Values.deploymentAnnotations }}
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
        app: istio-ingressgateway
        istio: ingressgateway
        istio.io/rev: asm-managed-stable
        
        
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



  Error Message:  KNV2009: failed to apply Deployment.apps, istio-ingressgateway/istio-ingressgateway: Deployment.apps "istio-ingressgateway" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"app":"istio-ingressgateway", "istio":"ingressgateway", "istio.io/rev":"asm-managed-stable"}: `selector` does not match template `labels`