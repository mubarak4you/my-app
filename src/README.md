#!/bin/bash

set -euo pipefail

export OCI_CLI_AUTH=instance_principal
oci ce cluster create-kubeconfig --cluster-id ${CLUSTER_ID}

timeout=420 # Maximum timeout in seconds
start_time=$(date +%s)
helmreleases="$(kubectl get helmreleases.helm.toolkit.fluxcd.io --namespace kube-system --no-headers)"

# Block to verify all helm releases have been pruned
while [[ $(eval "$helmreleases" | wc -l) -gt 0 ]]; do
  current_time=$(date +%s)
  elapsed_time=$((current_time - start_time))

  if [[ $elapsed_time -gt $timeout ]]; then
    echo "Timeout reached. Exiting..."
    exit 1
  fi

  sleep 5
  
  # New block to delete any remaining PVCs
  pvcs="$(kubectl get pvc --namespace kube-system --no-headers)"

  while [[ $(eval "$pvcs" | wc -l) -gt 0 ]]; do
    current_time=$(date +%s)
    elapsed_time=$((current_time - start_time))

    if [[ $elapsed_time -gt $timeout ]]; then
      echo "Timeout reached while deleting PVCs. Exiting..."
      exit 1
    fi

    echo "Deleting PVCs:"
    kubectl get pvc --namespace kube-system --no-headers | awk '{print $1}' | while read pvc; do
      echo "Deleting PVC: $pvc"
      kubectl delete pvc $pvc --namespace kube-system
    done

    sleep 5
  done
done
