[alabimu@10-74-129-4 ~]$ 
[alabimu@10-74-129-4 ~]$ kubectl run sysdig-cputest --image=go0v-vzdocker/containers/cicd/httpbin/2.6.0 --restart=Never --   sh -c 'for i in $(seq 1 20); do /bin/true; done; sleep 30'
Warning: [vzplatform-containerlimits-ceiling] container <sysdig-cputest> has no resource limits
Warning: [vzplatform-containerrequests-ceiling] container <sysdig-cputest> has no resource requests
Error from server (Forbidden): admission webhook "validation.gatekeeper.sh" denied the request: [vzsec-psp-allowedusers] Container sysdig-cputest is attempting to run without a required securityContext/runAsNonRoot or securityContext/runAsUser != 0
[vzec-psp-allowprivilegeescalationcontainer] Privilege escalation container is not allowed: sysdig-cputest
[vzplatform-required-containerresources] container <sysdig-cputest> does not have <{"cpu", "memory"}> limits defined
[vzplatform-required-containerresources] container <sysdig-cputest> does not have <{"cpu", "memory"}> requests defined
[vzsec-allowedrepos-system-user-ns] container <sysdig-cputest> has an invalid image repo <go0v-vzdocker/containers/cicd/httpbin/2.6.0>, allowed repos are ["us-docker.pkg.dev/vz-it-np-ivqv-dev-mldmdo-0", "us-east4-docker.pkg.dev/vz-it-np-ivqv-dev-mldmdo-0", "us-west1-docker.pkg.dev/vz-it-np-ivqv-dev-mldmdo-0", "oneartifactoryci-east.verizon.com", "oneartifactoryci-west.verizon.com", "gcr.io/gke-release/asm/proxyv2", "oneartifactoryci.verizon.com", "go0v-vzdocker.oneartifactoryprod.verizon.com"]
[vzsec-psp-capabilities] container <sysdig-cputest> is not dropping all required capabilities. Container must drop all of ["ALL"] or "ALL"
[vzsec-psp-readonlyrootfilesystem] only read-only root filesystem container is allowed: sysdig-cputest
[alabimu@10-74-129-4 ~]$ 
[alabimu@10-74-129-4 ~]$ 
[alabimu@10-74-129-4 ~]$ 
[alabimu@10-74-129-4 ~]$ 




