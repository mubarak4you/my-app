kubectl get daemonsets --all-namespaces -o custom-columns='NAMESPACE:.metadata.namespace,NAME:.metadata.name,PRIORITY:.spec.template.spec.priorityClassName'
