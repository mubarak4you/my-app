my values

resources:
  limits:
    cpu: 2000m
    memory: 1024Mi
  requests:
    cpu: 100m
    memory: 128Mi
serviceAccountName: istio-ingressgateway

imagePullSecret: go0v-vzdocker

_helpers.tpl
{{/*
Create the name of the service account to use
*/}}
{{- define "istio-ingressgateway.serviceAccountName" -}}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}

serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: istio-ingressgateway
  namespace: {{ .Values.namespace }}



got the error
    Code:           2004
      Error Message:  KNV2004: error in the helm-sync container: {"Msg":"unexpected error rendering chart, will retry","Err":"rendering helm chart: invoking helm: Pulled: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/dev/charts/istio-ingressgateway:1.0.1\nDigest: sha256:cd86cfebdb8480dac61342b9d93d5f8ec02e57a5e987f29efcf505785a63d59b\nError: template: istio-ingressgateway/templates/rolebinding.yaml:12:11: executing \"istio-ingressgateway/templates/rolebinding.yaml\" at \u003cinclude \"istio-ingressgateway.serviceAccountName\" .\u003e: error calling include: template: istio-ingressgateway/templates/_helpers.tpl:60:29: executing \"istio-ingressgateway.serviceAccountName\" at \u003c.Values.serviceAccount.name\u003e: nil pointer evaluating interface {}.name\n\nUse --debug flag to render out invalid YAML\n: exit status 1","Args":{}}