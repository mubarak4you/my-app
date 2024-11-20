#!/bin/bash

# Number of iterations for the load test
ITERATIONS=50
# Namespace to perform the test
NAMESPACE="rbac-load-test"

# Function to create roles and bindings
create_rbac_resources() {
    ROLE_NAME="test-role-$RANDOM"
    ROLEBINDING_NAME="test-rolebinding-$RANDOM"
    echo "Creating Role: $ROLE_NAME and RoleBinding: $ROLEBINDING_NAME in namespace $NAMESPACE"
    
    # Create a Role with read-only access to pods
    kubectl create role $ROLE_NAME --verb=get --verb=list --verb=watch --resource=pods -n $NAMESPACE
    
    # Create a RoleBinding for the Role to bind to the default service account
    kubectl create rolebinding $ROLEBINDING_NAME --role=$ROLE_NAME --serviceaccount=$NAMESPACE:default -n $NAMESPACE
}

# Function to delete roles and bindings
delete_rbac_resources() {
    ROLE_NAME=$1
    ROLEBINDING_NAME=$2
    echo "Deleting Role: $ROLE_NAME and RoleBinding: $ROLEBINDING_NAME in namespace $NAMESPACE"
    
    kubectl delete role $ROLE_NAME -n $NAMESPACE
    kubectl delete rolebinding $ROLEBINDING_NAME -n $NAMESPACE
}

# Ensure the namespace exists
kubectl create namespace $NAMESPACE || true

for ((i=1; i<=ITERATIONS; i++))
do
    echo "Iteration $i/$ITERATIONS"
    
    # Create resources
    create_rbac_resources
    ROLE_NAME="test-role-$RANDOM"
    ROLEBINDING_NAME="test-rolebinding-$RANDOM"

    # Give some time between operations (optional)
    sleep 0.5

    # Delete resources
    delete_rbac_resources $ROLE_NAME $ROLEBINDING_NAME
done

# Clean up namespace
kubectl delete namespace $NAMESPACE
echo "Load test completed."
