#!/bin/bash

# Variables
NAMESPACE="kube-system"
INGRESS_CLASS_NAME="grafana-oke-mbk-np-iad-go0v"
INGRESS_CLASS_PARAMS_NAME="platform-ingress"
BASE_HOST="grafana-oke-mbk-np-iad-go0v"
SERVICE_NAME="grafana"
TLS_SECRET_NAME="grafana-oke-mbk-np-iad-go0v"
NUMBER_OF_INGRESS=100  # Specify the number of ingress resources to create

# Function to delete resources
delete_resources() {
  echo "Deleting $NUMBER_OF_INGRESS ingress resources..."
  for i in $(seq 1 $NUMBER_OF_INGRESS); do
    INGRESS_NAME="grafana-ingress-$i"
    echo "Deleting Ingress: $INGRESS_NAME"
    kubectl delete ingress "$INGRESS_NAME" -n $NAMESPACE --ignore-not-found
  done

  echo "Deleting IngressClass: $INGRESS_CLASS_NAME"
  kubectl delete ingressclass "$INGRESS_CLASS_NAME" --ignore-not-found

  echo "Deleting IngressClassParameters: $INGRESS_CLASS_PARAMS_NAME"
  kubectl delete ingressclassparameters "$INGRESS_CLASS_PARAMS_NAME" -n $NAMESPACE --ignore-not-found

  echo "All resources have been deleted."
}

# Function to create resources
create_resources() {
  echo "Creating IngressClassParameters..."
  cat <<EOF | kubectl apply -f -
apiVersion: ingress.oraclecloud.com/v1beta1
kind: IngressClassParameters
metadata:
  name: $INGRESS_CLASS_PARAMS_NAME
  namespace: $NAMESPACE
  annotations:
    meta.helm.sh/release-name: oke-platform-ingress
    meta.helm.sh/release-namespace: $NAMESPACE
  labels:
    app.kubernetes.io/instance: oke-platform-ingress
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: oke-platform-ingress
    app.kubernetes.io/version: 1.0.0
spec:
  compartmentId: ocid1.compartment.oc1..aaaaaaaaro6irlkpiqdiilvjcx5gaohnce3vc2khri57fcaymyqps5mddvaq
  isPrivate: true
  maxBandwidthMbps: 400
  minBandwidthMbps: 100
  subnetId: ocid1.subnet.oc1.iad.aaaaaaaa5qhbqidcyflys6gmujxixnqa3ai2eq55sxrpmdbabtc5746sgnba
EOF

  echo "Creating IngressClass..."
  cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: $INGRESS_CLASS_NAME
  annotations:
    meta.helm.sh/release-name: oke-platform-ingress
    meta.helm.sh/release-namespace: $NAMESPACE
    oci-native-ingress.oraclecloud.com/id: ocid1.loadbalancer.oc1.iad.aaaaaaaakxwmblyh2bcskccvasrnacinnlw6occ6wdkt325ldup3xsvskgfq
  labels:
    app.kubernetes.io/instance: oke-platform-ingress
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: oke-platform-ingress
    app.kubernetes.io/version: 1.0.0
spec:
  controller: oci.oraclecloud.com/native-ingress-controller
  parameters:
    apiGroup: ingress.oraclecloud.com
    kind: IngressClassParameters
    name: $INGRESS_CLASS_PARAMS_NAME
    namespace: $NAMESPACE
    scope: Namespace
EOF

  echo "Creating $NUMBER_OF_INGRESS ingress resources..."
  for i in $(seq 1 $NUMBER_OF_INGRESS); do
    INGRESS_NAME="grafana-ingress-$i"
    HOST="${BASE_HOST}-$i.ebiz.verizon.com"
    echo "Creating Ingress: $INGRESS_NAME with Host: $HOST"
    cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: $INGRESS_NAME
  namespace: $NAMESPACE
  annotations:
    meta.helm.sh/release-name: oke-platform-ingress
    meta.helm.sh/release-namespace: $NAMESPACE
    oci-native-ingress.oraclecloud.com/backend-tls-enabled: "false"
    oci-native-ingress.oraclecloud.com/healthcheck-force-plaintext: "true"
    oci-native-ingress.oraclecloud.com/healthcheck-path: /api/health
    oci-native-ingress.oraclecloud.com/healthcheck-protocol: HTTP
    oci-native-ingress.oraclecloud.com/https-listener-port: "443"
    oci-native-ingress.oraclecloud.com/protocol: HTTP2
  labels:
    app.kubernetes.io/instance: oke-platform-ingress
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: oke-platform-ingress
    app.kubernetes.io/version: 1.0.0
spec:
  ingressClassName: $INGRESS_CLASS_NAME
  rules:
  - host: $HOST
    http:
      paths:
      - backend:
          service:
            name: $SERVICE_NAME
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - $HOST
    secretName: $TLS_SECRET_NAME
EOF
  done

  echo "All resources have been created successfully."
}

# Main script logic
if [[ "$1" == "--delete" ]]; then
  delete_resources
else
  create_resources
fi
