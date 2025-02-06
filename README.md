[alabimu@10-74-128-76 filestore]$ cat *
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reader
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reader
  template:
    metadata:
      labels:
        app: reader
    spec:
      securityContext:
        fsGroup: 1001
        supplementalGroups: [1001]
      containers:
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: fileserver
          mountPath: /usr/share/nginx/html
          readOnly: true
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
      volumes:
      - name: fileserver
        persistentVolumeClaim:
          claimName: fileserver

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
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fileserver-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: csi-gce-filestore-cmek
  csi:
    driver: filestore.csi.storage.gke.io
    volumeHandle: "projects/vz-it-np-exhv-sharedvpc-228116/locations/us-east4/instances/gke-sample/volumes/vol1"



[alabimu@10-74-128-76 filestore]$ kubectl create -f .
deployment.apps/reader created
persistentvolume/fileserver-pv created
persistentvolumeclaim/fileserver created
[alabimu@10-74-128-76 filestore]$ kubectl get deployments
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
nginx    1/1     1            1           24d
reader   0/1     0            0           3s
[alabimu@10-74-128-76 filestore]$ kubectl get pvc 
NAME                    STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS                  VOLUMEATTRIBUTESCLASS   AGE
fileserver              Pending                                      csi-gce-filestore-cmek        <unset>                 7s
test-store-test-set-0   Pending                                      csi-gce-filestore-test-cmek   <unset>                 357d
[alabimu@10-74-128-76 filestore]$ kubectl get pv
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS             VOLUMEATTRIBUTESCLASS   REASON   AGE
fileserver-pv   10Gi       RWX            Retain           Available           csi-gce-filestore-cmek   <unset>                          10s
[alabimu@10-74-128-76 filestore]$ kubectl describe deployments reader
Name:                   reader
Namespace:              default
CreationTimestamp:      Thu, 06 Feb 2025 16:09:37 -0500
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
  Normal  ScalingReplicaSet  15s   deployment-controller  Scaled up replica set reader-64558b8c86 to 1
  
  
  
  
  
  