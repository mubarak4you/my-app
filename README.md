for resource in k8sallowedrepos k8singresshttps k8spspallowedusers k8spspallowprivilegeescalationcontainer \
k8spspapparmor k8spspcapabilities k8spspcapabilitiesimageexception k8spsphostfilesystem \
k8spsphostfilesystemimageexception k8spsphostnamespace k8spspnetnetworkingports k8spspprivilegedcontainer \
k8spspreadonlyrootfilesystem k8spspseccomp k8spspvolumetypes k8srequiredlabels \
k8sserviceaccountsecrets vzsecanonymous vzsecgatewayconfig vzseck8srbac vzsecrbacprivilegedverb; do

  echo -e "\n🔍 Checking: $resource"
  kubectl get "$resource" -A -o json 2>/dev/null | jq -r '
    .items[] | select(.status.violations != null) |
    "Name: \(.metadata.name)\nNamespace: \(.metadata.namespace // "cluster-scoped")\nViolations:\n\(.status.violations)\n---"
  '
done
