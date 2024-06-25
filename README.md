resource "kubernetes_secret_v1" "go0v_vzdocker" {
  metadata {
    name      = "go0v-vzdocker-config"
    namespace = "kube-system"
    annotations = {
      "reflector.v1.k8s.emberstack.com/reflection-allowed"      = "true"
      "reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces" = "cert-manager,flux-system,gatekeeper-system,gitlab-runner-system"
      "reflector.v1.k8s.emberstack.com/reflection-auto-enabled" = "true"
    }
  }
  data = {
    ".dockerconfigjson" = base64decode(var.go0v_vzdocker_config)
  }
  type = "kubernetes.io/dockerconfigjson"
}