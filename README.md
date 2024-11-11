#!/bin/bash

# Number of secrets to create
NUM_SECRETS=5

# Prefix for the secret names
SECRET_PREFIX="reflected-secret"

# Allowed namespaces for reflection
ALLOWED_NAMESPACES="ns1,ns2"

# Function to create a secret with annotations
create_reflected_secret() {
  local name="${SECRET_PREFIX}-$1"
  
  # Step 1: Create the secret
  kubectl create secret generic "$name" \
    --from-literal="key1=$(echo -n 'value1' | base64)" \
    --dry-run=client -o yaml | kubectl apply -f -
  
  # Step 2: Add annotations to the secret
  kubectl annotate secret "$name" \
    reflector.v1.k8s.emberstack.com/reflection-allowed="true" \
    reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces="$ALLOWED_NAMESPACES"
}

# Loop to create multiple secrets
for i in $(seq 1 $NUM_SECRETS); do
  echo "Creating reflected secret ${SECRET_PREFIX}-${i}"
  create_reflected_secret "$i"
done

echo "All reflected secrets created."
