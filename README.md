apiVersion: configsync.gke.io/v1beta1
kind: RootSync
metadata:
  name: istio-ingressgateway
  namespace: config-management-system
spec:
  sourceFormat: unstructured
  sourceType: helm
  helm:
    repo: oci://go0v-vzdocker.oneartifactoryprod.verizon.com/containers/charts
    chart: istio-ingressgateway
    version: "1.0.0"
    releaseName: istio-ingressgateway
    namespace: asm-ingressgateway  # This only affects Helm templates with {{ .Release.Namespace }}
    deployNamespace: asm-ingressgateway  # ✅ Forces namespace for all resources
    auth: token
    secretRef:
      name: rootsync-artifactory
    values:
      image:
        repository: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/istio-ingressgateway
        tag: 1.0.0
      imagePullSecrets:
        - name: go0v-vzdocker-config
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          drop:
            - ALL
        privileged: false
        readOnlyRootFilesystem: false
        runAsGroup: 1000
        runAsNonRoot: true
        runAsUser: 1000
      resources:
        limits:
          cpu: "2000m"
          memory: "1024Mi"
        requests:
          cpu: "100m"
          memory: "128Mi"
      serviceAccountName: istio-ingressgateway
