kubectl get daemonsets -A -o json | jq -r '.items[] | "\(.metadata.namespace) \(.metadata.name) \(.spec.template.spec.containers[]?.resources.requests.cpu // "none")"' | column -t
