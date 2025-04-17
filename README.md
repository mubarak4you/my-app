[alabimu@63-22-141-67 ~]$ kubectl run sysdig-memtest --image=go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/reflector:7.1.262 --restart=Never -- \
>   sh -c 'for i in $(seq 1 5); do echo "testdata $i" > /tmp/file-$i.log; done; sleep 60'
Warning: [k8s-psp-allowed-users-fsgroup] Container sysdig-memtest is attempting to run without a required securityContext/fsGroup. Allowed fsGroup: {"ranges": [{"max": 65535, "min": 1}], "rule": "MustRunAs"}
Warning: [k8s-psp-allowed-users-supplemental-groups] Container sysdig-memtest is attempting to run without a required securityContext/supplementalGroups. Allowed supplementalGroups: {"ranges": [{"max": 65535, "min": 1}], "rule": "MustRunAs"}
Warning: [k8s-psp-seccomp] Seccomp profile 'not configured' is not allowed for container 'sysdig-memtest'. Found at: no explicit profile found. Allowed profiles: {"RuntimeDefault", "docker/default", "runtime/default"}
Error from server (Forbidden): admission webhook "validation.gatekeeper.sh" denied the request: [k8s-psp-allowed-users-runasuser] Container sysdig-memtest is attempting to run without a required securityContext/runAsNonRoot or securityContext/runAsUser != 0
[k8s-psp-allow-privilege-escalation-container] Privilege escalation container is not allowed: sysdig-memtest
[k8s-psp-capabilities] container <sysdig-memtest> is not dropping all required capabilities. Container must drop all of ["all"] or "ALL"
[alabimu@63-22-141-67 ~]$ 
[alabimu@63-22-141-67 ~]$ 
[alabimu@63-22-141-67 ~]$ kubectl get pods -n default
No resources found in default namespace.
(reverse-i-search)`get p': kubectl ^Ct pods -n kube-system | grep sys
[alabimu@63-22-141-67 ~]$ kubectl get pods -A | grep reflc
[alabimu@63-22-141-67 ~]$ kubectl get pods -A | grep refl
kube-system            reflector-5798754d44-vlhf2                                      1/1     Running     0            9h