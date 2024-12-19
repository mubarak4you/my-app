#!/bin/bash

# Number of Ingresses to create
NUM_INGRESSES=10

# Base name and host
BASE_NAME="agrafana-ingress"
BASE_HOST="oke-mbkhpa-np-iad-go0v.ebiz.verizon.com"

# Namespace for the Ingress
NAMESPACE="kube-system"

# Output directory
OUTPUT_DIR="./ingresses"
mkdir -p "$OUTPUT_DIR"

for i in $(seq 1 $NUM_INGRESSES); do
  # Generate unique name and host
  NAME="${i}-${BASE_NAME}"
  HOST="${i}-${BASE_HOST}"

  # Create Ingress YAML
  cat <<EOF > "${OUTPUT_DIR}/${NAME}.yaml"
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
  name: $NAME
  namespace: $NAMESPACE
spec:
  ingressClassName: grafana-oke-mbkhpa-np-iad-go0v
  rules:
  - host: $HOST
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
    - $HOST
    secretName: grafana-oke-mbkhpa-np-iad-go0v
EOF

  echo "Created ${OUTPUT_DIR}/${NAME}.yaml"
done
