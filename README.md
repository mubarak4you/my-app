		
gke-sample
vol1	Feb 6, 2025, 2:06:04 PM	ENTERPRISE	us-east4	NFSv3	192.168.224.66	1 TiB	


give me a pv and pvc and pod that gets attatched to the fileshare instance on my gke cluster.

storageclass name is csi-gce-filestore-cmek

use as reference
https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/filestore-csi-driver

pvc should be storage: 10Gi

include

    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
          - "ALL"
      privileged: false
      readOnlyRootFilesystem: true
      runAsNonRoot: true
      seccompProfile:
        type: RuntimeDefault
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 500m
        memory: 256Mi
        

image on the pod should be
go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/apache:2.4.58