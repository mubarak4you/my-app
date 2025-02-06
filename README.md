[alabimu@10-74-128-76 filestore]$ kubectl get deployments
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
nginx    1/1     1            1           24d
reader   0/1     0            0           83s
[alabimu@10-74-128-76 filestore]$ kubectl describe deployments reader
Name:                   reader
Namespace:              default
CreationTimestamp:      Thu, 06 Feb 2025 12:45:12 -0500
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=reader
Replicas:               1 desired | 0 updated | 0 total | 0 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=reader
  Containers:
   nginx:
    Image:           nginx:stable-alpine
    Port:            80/TCP
    Host Port:       0/TCP
    SeccompProfile:  RuntimeDefault
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        100m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /usr/share/nginx/html from fileserver (ro)
  Volumes:
   fileserver:
    Type:          PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:     fileserver
    ReadOnly:      false
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type             Status  Reason
  ----             ------  ------
  Progressing      True    NewReplicaSetCreated
  Available        False   MinimumReplicasUnavailable
  ReplicaFailure   True    FailedCreate
OldReplicaSets:    <none>
NewReplicaSet:     reader-64558b8c86 (0/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  87s   deployment-controller  Scaled up replica set reader-64558b8c86 to 1
[alabimu@10-74-128-76 filestore]$ kubectl get pvc 
NAME                    STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS                  VOLUMEATTRIBUTESCLASS   AGE
fileserver              Pending                                      csi-gce-filestore-cmek        <unset>                 19m
test-store-test-set-0   Pending                                      csi-gce-filestore-test-cmek   <unset>                 357d
[alabimu@10-74-128-76 filestore]$ kubectl describe pvc fileserver
Name:          fileserver
Namespace:     default
StorageClass:  csi-gce-filestore-cmek
Status:        Pending
Volume:        
Labels:        <none>
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      
Access Modes:  
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                Age                   From                         Message
  ----    ------                ----                  ----                         -------
  Normal  WaitForFirstConsumer  4m15s (x62 over 19m)  persistentvolume-controller  waiting for first consumer to be created before binding
[alabimu@10-74-128-76 filestore]$ 
[alabimu@10-74-128-76 filestore]$ 
[alabimu@10-74-128-76 filestore]$ 
[alabimu@10-74-128-76 filestore]$ kubectl get deployments
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
nginx    1/1     1            1           24d
reader   0/1     0            0           114s
[alabimu@10-74-128-76 filestore]$ kubectl describe deployments reader
Name:                   reader
Namespace:              default
CreationTimestamp:      Thu, 06 Feb 2025 12:45:12 -0500
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=reader
Replicas:               1 desired | 0 updated | 0 total | 0 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=reader
  Containers:
   nginx:
    Image:           nginx:stable-alpine
    Port:            80/TCP
    Host Port:       0/TCP
    SeccompProfile:  RuntimeDefault
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        100m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /usr/share/nginx/html from fileserver (ro)
  Volumes:
   fileserver:
    Type:          PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:     fileserver
    ReadOnly:      false
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type             Status  Reason
  ----             ------  ------
  Progressing      True    NewReplicaSetCreated
  Available        False   MinimumReplicasUnavailable
  ReplicaFailure   True    FailedCreate
OldReplicaSets:    <none>
NewReplicaSet:     reader-64558b8c86 (0/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  2m2s  deployment-controller  Scaled up replica set reader-64558b8c86 to 1
(failed reverse-i-search)`get stor': kubectl ^Ct pvc 
[alabimu@10-74-128-76 filestore]$ kubectl get storageclass
NAME                              PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
csi-gce-balanced-cmek (default)   pd.csi.storage.gke.io          Delete          WaitForFirstConsumer   true                   482d
csi-gce-filestore-cmek            filestore.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   309d
csi-gce-pd-cmek (default)         pd.csi.storage.gke.io          Delete          WaitForFirstConsumer   true                   647d
csi-gce-ssd-cmek                  pd.csi.storage.gke.io          Delete          WaitForFirstConsumer   true                   647d
enterprise-multishare-rwx         filestore.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   647d
enterprise-rwx                    filestore.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   647d
premium-rwo                       pd.csi.storage.gke.io          Delete          WaitForFirstConsumer   true                   371d
premium-rwx                       filestore.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   647d
standard                          kubernetes.io/gce-pd           Delete          Immediate              true                   647d
standard-rwo                      pd.csi.storage.gke.io          Delete          WaitForFirstConsumer   true                   371d
standard-rwx                      filestore.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   371d
zonal-rwx                         filestore.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   386d
[alabimu@10-74-128-76 filestore]$ 
[alabimu@10-74-128-76 filestore]$ 
[alabimu@10-74-128-76 filestore]$ cat pvc.yaml 
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: fileserver
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: csi-gce-filestore-cmek
  resources:
    requests:
      storage: 10Gi
[alabimu@10-74-128-76 filestore]$ kubectl get pvc 
NAME                    STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS                  VOLUMEATTRIBUTESCLASS   AGE
fileserver              Pending                                      csi-gce-filestore-cmek        <unset>                 22m
test-store-test-set-0   Pending                                      csi-gce-filestore-test-cmek   <unset>                 357d
[alabimu@10-74-128-76 filestore]$ kubectl get pv
No resources found
