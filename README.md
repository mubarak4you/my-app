[alabimu@10-74-128-76 ~]$ kubectl create -f new.yaml 
Warning: [vzplatform-containerlimits-ceiling] container <app> has no resource limits
Error from server (Forbidden): error when creating "new.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [vzplatform-required-containerresources] container <app> does not have <{"cpu", "memory"}> limits defined