apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    meta.helm.sh/release-name: oke-platform-ingress
    meta.helm.sh/release-namespace: kube-system
    oci-native-ingress.oraclecloud.com/backend-tls-enabled: "false"
    oci-native-ingress.oraclecloud.com/healthcheck-force-plaintext: "true"
    oci-native-ingress.oraclecloud.com/healthcheck-path: /api/health
    oci-native-ingress.oraclecloud.com/healthcheck-protocol: HTTP
    oci-native-ingress.oraclecloud.com/https-listener-port: "443"
    oci-native-ingress.oraclecloud.com/protocol: HTTP2
  creationTimestamp: "2024-12-10T06:07:48Z"
  finalizers:
  - oci.oraclecloud.com/ingress-controller-protection
  generation: 1
  labels:
    app.kubernetes.io/instance: oke-platform-ingress
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: oke-platform-ingress
    app.kubernetes.io/version: 1.0.0
    helm.toolkit.fluxcd.io/name: oke-platform-ingress
    helm.toolkit.fluxcd.io/namespace: kube-system
  name: grafana-ingress
  namespace: kube-system
  resourceVersion: "5382"
  uid: fdd4d7eb-2a7c-4040-8635-a054ff5c1df7
spec:
  ingressClassName: grafana-oke-mbk-np-iad-go0v
  rules:
  - host: grafana-oke-mbk-np-iad-go0v.ebiz.verizon.com
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
    - grafana-oke-mbk-np-iad-go0v.ebiz.verizon.com
    secretName: grafana-oke-mbk-np-iad-go0v
status:
  loadBalancer:
    ingress:
    - ip: 63.22.133.140





apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  annotations:
    meta.helm.sh/release-name: oke-platform-ingress
    meta.helm.sh/release-namespace: kube-system
    oci-native-ingress.oraclecloud.com/id: ocid1.loadbalancer.oc1.iad.aaaaaaaakxwmblyh2bcskccvasrnacinnlw6occ6wdkt325ldup3xsvskgfq
  creationTimestamp: "2024-12-10T06:07:48Z"
  finalizers:
  - oci.oraclecloud.com/ingress-controller-protection
  generation: 1
  labels:
    app.kubernetes.io/instance: oke-platform-ingress
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: oke-platform-ingress
    app.kubernetes.io/version: 1.0.0
    helm.toolkit.fluxcd.io/name: oke-platform-ingress
    helm.toolkit.fluxcd.io/namespace: kube-system
  name: grafana-oke-mbk-np-iad-go0v
  resourceVersion: "5365"
  uid: fcefbaed-1ea9-4c1d-bee5-ffdf1168558c
spec:
  controller: oci.oraclecloud.com/native-ingress-controller
  parameters:
    apiGroup: ingress.oraclecloud.com
    kind: ingressclassparameters
    name: platform-ingress
    namespace: kube-system
    scope: Namespace




apiVersion: ingress.oraclecloud.com/v1beta1
kind: IngressClassParameters
metadata:
  annotations:
    meta.helm.sh/release-name: oke-platform-ingress
    meta.helm.sh/release-namespace: kube-system
  creationTimestamp: "2024-12-10T06:07:48Z"
  generation: 1
  labels:
    app.kubernetes.io/instance: oke-platform-ingress
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: oke-platform-ingress
    app.kubernetes.io/version: 1.0.0
    helm.toolkit.fluxcd.io/name: oke-platform-ingress
    helm.toolkit.fluxcd.io/namespace: kube-system
  name: platform-ingress
  namespace: kube-system
  resourceVersion: "5369"
  uid: 7e13c7c7-61bc-4c1d-9a44-f416dd292870
spec:
  compartmentId: ocid1.compartment.oc1..aaaaaaaaro6irlkpiqdiilvjcx5gaohnce3vc2khri57fcaymyqps5mddvaq
  isPrivate: true
  maxBandwidthMbps: 400
  minBandwidthMbps: 100
  subnetId: ocid1.subnet.oc1.iad.aaaaaaaa5qhbqidcyflys6gmujxixnqa3ai2eq55sxrpmdbabtc5746sgnba