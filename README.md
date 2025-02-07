apiVersion: v1
kind: PersistentVolume
metadata:
  name: gke-filestore-pv
spec:
  capacity:
    storage: 1Ti
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: csi-gce-filestore-cmek
  csi:
    driver: filestore.csi.storage.gke.io
    volumeHandle: "projects/vz-it-np-go0v-dev-gketst-0/locations/us-east4/instances/gke-sample"
    volumeAttributes:
      ip: "192.168.224.66"
      volume: "vol1"
      protocol: "nfs"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gke-filestore-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: csi-gce-filestore-cmek
---
apiVersion: v1
kind: Pod
metadata:
  name: filestore-client-pod
spec:
  containers:
  - name: apache-container
    image: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/apache:2.4.58
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
    volumeMounts:
    - mountPath: "/mnt/data"
      name: filestore-volume
  volumes:
  - name: filestore-volume
    persistentVolumeClaim:
      claimName: gke-filestore-pvc
