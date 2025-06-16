for ns in $(kubectl get helmrelease --all-namespaces -o jsonpath='{.items[*].metadata.namespace}' | tr ' ' '\n' | sort -u); do
  for name in $(kubectl get helmrelease -n "$ns" -o jsonpath='{.items[*].metadata.name}'); do
    echo "Checking release: $ns/$name"
    helm get values "$name" -n "$ns" 2>/dev/null | grep ETOKE-438 && echo "↳ Found in $ns/$name"
  done
done
