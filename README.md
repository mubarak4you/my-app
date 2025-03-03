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


resources:
  limits:
    cpu: 2000m
    memory: 1024Mi
  requests:
    cpu: 100m
    memory: 128Mi
serviceAccountName: istio-ingressgateway

imagePullSecret: go0v-vzdocker