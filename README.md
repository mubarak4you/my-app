#!/bin/bash

# Number of secrets to create
NUM_SECRETS=5

# Prefix for the secret names
SECRET_PREFIX="reflected-secret"

# Allowed namespaces for reflection
ALLOWED_NAMESPACES="ns1,ns2"

# Function to create a YAML manifest for a secret with annotations
create_reflected_secret_manifest() {
  local name="${SECRET_PREFIX}-$1"

  # Generate YAML manifest
  cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: $name
  annotations:
    reflector.v1.k8s.emberstack.com/reflection-allowed: "true"
    reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces: "$ALLOWED_NAMESPACES"
    reflector.v1.k8s.emberstack.com/reflection-auto-enabled: "true"
type: Opaque
data:
  key1: $(echo -n 'value1' | base64)
EOF
}

# Loop to create multiple secrets using the YAML manifest approach
for i in $(seq 1 $NUM_SECRETS); do
  echo "Creating reflected secret ${SECRET_PREFIX}-${i} using YAML manifest"
  create_reflected_secret_manifest "$i"
done

echo "All reflected secrets created with annotations."
