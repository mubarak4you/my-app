apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    oci-native-ingress.oraclecloud.com/backend-tls-enabled: "false"
    oci-native-ingress.oraclecloud.com/healthcheck-force-plaintext: "true"
    oci-native-ingress.oraclecloud.com/healthcheck-path: /api/health
    oci-native-ingress.oraclecloud.com/healthcheck-protocol: HTTP
    oci-native-ingress.oraclecloud.com/https-listener-port: "443"
    oci-native-ingress.oraclecloud.com/protocol: HTTP2
  finalizers:
  - oci.oraclecloud.com/ingress-controller-protection
  generation: 1
  labels:
    app.kubernetes.io/instance: oke-platform-ingress
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: oke-platform-ingress
    app.kubernetes.io/version: 1.0.0
  name: agrafana-ingress
  namespace: kube-system
spec:
  ingressClassName: grafana-oke-mbkhpa-np-iad-go0v
  rules:
  - host: agrafana-oke-mbkhpa-np-iad-go0v.ebiz.verizon.com
    http:
      paths:
      - backend:
          service:
            name: grafana
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - agrafana-oke-mbkhpa-np-iad-go0v.ebiz.verizon.com
    secretName: grafana-oke-mbkhpa-np-iad-go0v