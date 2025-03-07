{{/*
The initContainer image to use
*/}}
{{- define "istio-ingressgateway.initContainerImage" -}}
{{- printf "%s:%s" .Values.initContainer.image.repository .Values.initContainer.image.tag }}
{{- end }}


initContainer:
  image:
    repository: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/kubectl
    tag: 1.30.5


initContainers:
  - image: {{ include "istio-ingressgateway.initContainerImage" . }}
    name: init
