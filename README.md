{{/*
Create the name of the service account to use
*/}}
{{- define "istio-ingressgateway.serviceAccountName" -}}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}