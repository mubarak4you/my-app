#!/bin/bash

# Purpose of this script is to wait for Helm Releases in the kube-system namespace
# to be completely deleted from the cluster before terraform deletes the cluster.

set -euo pipefail

export OCI_CLI_AUTH=instance_principal
oci ce cluster create-kubeconfig --cluster-id ${CLUSTER_ID}

timeout=240 # Maximum timeout in seconds
start_time=$(date +%s)

while true; do
  helmreleases=$(kubectl get helmreleases.helm.toolkit.fluxcd.io --namespace kube-system --no-headers)
  if [[ -z "$helmreleases" ]]; then
    echo "No Helm releases found. Cleaning up PVCs and exiting..."
    pvcs=$(kubectl get pvc --namespace kube-system -o jsonpath='{.items[*].metadata.name}')
    for pvc in $pvcs; do
      echo "Deleting PVC: $pvc"
      kubectl delete pvc "$pvc" --namespace kube-system
    done
    exit 0
  else
    echo "Helm releases found:"
    echo "$helmreleases"
  fi

  current_time=$(date +%s)
  elapsed_time=$((current_time - start_time))

  if [[ $elapsed_time -gt $timeout ]]; then
    echo "Timeout reached. Exiting..."
    exit 1
  fi

  sleep 5
done
