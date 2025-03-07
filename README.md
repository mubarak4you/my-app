{{/*
The istio-proxy container image to use
*/}}
{{- define "istio-ingressgateway.proxyImage" -}}
{{- printf "%s:%s" .Values.proxy.image.repository .Values.proxy.image.tag }}
{{- end }}


proxy:
  image:
    repository: auto
    tag: latest


containers:
  - name: istio-proxy
    image: {{ include "istio-ingressgateway.proxyImage" . }}
