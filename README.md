[alabimu@10-74-129-4 ~]$ kubectl get daemonsets -A -o json | jq -r '
>   .items[] | 
>   {
>     ns: .metadata.namespace,
>     name: .metadata.name,
>     total_cpu: (
>       [.spec.template.spec.containers[].resources.requests.cpu // "0m"]
>       | map(
>           sub("m$"; "") 
>           | if test("^[0-9]+$") then tonumber else 0 end
>         )
>       | add
>     )
>   } 
>   | "\(.ns) \(.name) \(.total_cpu)m"
> ' | column -t


NODE_NAME=<your-node-name>
kubectl get pods --all-namespaces -o json \
  | jq -r --arg NODE "$NODE_NAME" '
    .items[]
    | select(.spec.nodeName == $NODE)
    | select(.metadata.ownerReferences[]?.kind == "DaemonSet")
    | {
        ns: .metadata.namespace,
        name: .metadata.name,
        ds: .metadata.ownerReferences[0].name,
        cpu: (
          [.spec.containers[].resources.requests.cpu // "0m"]
          | map(
              sub("m$"; "") 
              | if test("^[0-9]+$") then tonumber else 0 end
            )
          | add
        )
      }
    | "\(.ns) \(.ds) \(.cpu)m"
  ' | column -t



NODE_NAME=<your-node-name>
kubectl get pods --all-namespaces -o json \
  | jq -r --arg NODE "$NODE_NAME" '
    [.items[]
     | select(.spec.nodeName == $NODE)
     | select(.metadata.ownerReferences[]?.kind == "DaemonSet")
     | [.spec.containers[].resources.requests.cpu // "0m"]
     | map(
         sub("m$"; "") 
         | if test("^[0-9]+$") then tonumber else 0 end
       )
     | add] 
    | add
  ' | awk '{print $1 "m"}'



NODE_NAME=<your-node-name>
kubectl get pods --all-namespaces -o json \
  | jq -r --arg NODE "$NODE_NAME" '
    [.items[]
     | select(.spec.nodeName == $NODE)
     | select(.metadata.ownerReferences[]?.kind == "DaemonSet")
     | [.spec.containers[].resources.requests.cpu // "0m"]
     | map(
         sub("m$"; "") 
         | if test("^[0-9]+$") then tonumber else 0 end
       )
     | add] 
    | add
  ' | awk '{printf "Total CPU: %sm (%.3f cores)\n", $1, $1 / 1000}'

