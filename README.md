#!/bin/bash

# Number of iterations for the stress test
ITERATIONS=50

# Temporary RBACDefinition YAML file
TEMP_RBAC_FILE="temp_rbac_definition.yaml"

# Namespaces to be created
NAMESPACES=("api" "web")

# Function to create namespaces
create_namespaces() {
  echo "Creating namespaces..."
  for ns in "${NAMESPACES[@]}"; do
    kubectl create namespace $ns || echo "Namespace $ns already exists"
  done
}

# Function to delete namespaces after the test
delete_namespaces() {
  echo "Deleting namespaces..."
  for ns in "${NAMESPACES[@]}"; do
    kubectl delete namespace $ns || echo "Failed to delete namespace $ns"
  done
}

# RBACDefinition template
generate_rbac_definition() {
  cat <<EOF > $TEMP_RBAC_FILE
apiVersion: rbacmanager.reactiveops.io/v1beta1
kind: RBACDefinition
metadata:
  name: user-access-$1
rbacBindings:
  - name: user-$1
    subjects:
      - kind: User
        name: user$1@example.com
    roleBindings:
      - namespace: api
        clusterRole: view
      - namespace: web
        clusterRole: edit
EOF
}

# Create required namespaces
create_namespaces

# Stress test loop
for ((i=1; i<=ITERATIONS; i++)); do
  echo "Iteration $i/$ITERATIONS"

  # Generate a unique RBACDefinition for this iteration
  generate_rbac_definition $i

  # Apply the RBACDefinition
  kubectl apply -f $TEMP_RBAC_FILE

  # Give some time for the RBAC manager to process (optional)
  sleep 0.5

  # Delete the RBACDefinition
  kubectl delete -f $TEMP_RBAC_FILE
done

# Cleanup
rm -f $TEMP_RBAC_FILE

# Delete namespaces after the test
delete_namespaces

echo "RBAC stress test completed."
