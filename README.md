for kind in $(kubectl get constrainttemplates -o jsonpath='{.items[*].spec.names.kind}'); do
  echo "Checking $kind...";
  kubectl get $kind -A -o json | jq -r '.items[] | select(.status.violations != null) | "Kind: '"$kind"'\nName: \(.metadata.name)\nNamespace: \(.metadata.namespace // "cluster-scoped")\nViolations:\n\(.status.violations)\n---"'
done


