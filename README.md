{{- define "istio-ingressgateway.serviceAccountName" -}}
{{- if .Values.serviceAccount }}
{{- default "default" .Values.serviceAccount.name }}
{{- else }}
default
{{- end }}
{{- end }}
