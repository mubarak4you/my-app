#!/bin/bash

# Number of iterations for the stress test
ITERATIONS=50

# Number of bindings per RBACDefinition to increase memory usage
BINDINGS_PER_DEFINITION=50

# Temporary RBACDefinition YAML file
TEMP_RBAC_FILE="temp_rbac_definition.yaml"

# Namespaces to be created
NAMESPACES=()
for i in $(seq 1 $BINDINGS_PER_DEFINITION); do
  NAMESPACES+=("test-ns-$i")
done

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

# RBACDefinition template generator with multiple bindings
generate_rbac_definition() {
  local index=$1
  cat <<EOF > $TEMP_RBAC_FILE
apiVersion: rbacmanager.reactiveops.io/v1beta1
kind: RBACDefinition
metadata:
  name: user-access-$index
rbacBindings:
EOF

  for i in $(seq 1 $BINDINGS_PER_DEFINITION); do
    cat <<EOF >> $TEMP_RBAC_FILE
  - name: user-$index-$i
    subjects:
      - kind: User
        name: user$index-$i@example.com
    roleBindings:
      - namespace: test-ns-$i
        clusterRole: view
      - namespace: test-ns-$i
        clusterRole: edit
EOF
  done
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
