#!/bin/bash

set -euo pipefail

export OCI_CLI_AUTH=instance_principal
oci ce cluster create-kubeconfig --cluster-id ${CLUSTER_ID}

timeout=420 # Maximum timeout in seconds
start_time=$(date +%s)

helmreleases=$(kubectl get helmreleases.helm.toolkit.fluxcd.io --namespace kube-system --no-headers)

while [[ $(eval "echo $helmreleases | wc -l") -gt 0 ]]; do
    current_time=$(date +%s)
    elapsed_time=$((current_time - start_time))

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

    if [[ $elapsed_time -gt $timeout ]]; then
        echo "Timeout reached. Exiting..."
        exit 1
    fi

    sleep 5
    helmreleases=$(kubectl get helmreleases.helm.toolkit.fluxcd.io --namespace kube-system --no-headers)
done
