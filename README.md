printf "%-35s %-20s %s\n" "NAME" "ENFORCEMENT-ACTION" "TOTAL-VIOLATIONS"
for resource in k8sallowedrepos k8singresshttps k8spspallowedusers k8spspallowprivilegeescalationcontainer \
k8spspapparmor k8spspcapabilities k8spspcapabilitiesimageexception k8spsphostfilesystem \
k8spsphostfilesystemimageexception k8spsphostnamespace k8spspnetnetworkingports k8spspprivilegedcontainer \
k8spspreadonlyrootfilesystem k8spspseccomp k8spspvolumetypes k8srequiredlabels \
k8sserviceaccountsecrets vzsecanonymous vzsecgatewayconfig vzseck8srbac vzsecrbacprivilegedverb; do

  kubectl get "$resource" -A -o json 2>/dev/null | jq -r '
    .items[] | "\(.metadata.name)\t\(.spec.enforcementAction)\t\(.status.totalViolations // 0)"
  ' | awk -F'\t' '{printf "%-35s %-20s %s\n", $1, $2, $3}'
done
