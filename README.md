
get priority class for all pods in a node

kubectl get pods --all-namespaces --field-selector spec.nodeName=<NODE_NAME> -o json | jq -r '
  .items[] | 
  "\(.metadata.namespace)\t\(.metadata.name)\t\(.spec.priorityClassName // "none")"
'


