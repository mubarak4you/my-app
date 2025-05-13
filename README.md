kubectl get pods --all-namespaces -o json \
  | jq '.items[] | {namespace: .metadata.namespace, name: .metadata.name, scheduler: .spec.schedulerName}'


kubectl get pods -n <namespace> -o json \
  | jq '.items[] | {name: .metadata.name, scheduler: .spec.schedulerName}'


kubectl get pods --all-namespaces --field-selector spec.nodeName=<node-name> -o json \
  | jq '.items[] | {namespace: .metadata.namespace, name: .metadata.name, scheduler: .spec.schedulerName}'


