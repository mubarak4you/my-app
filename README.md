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
