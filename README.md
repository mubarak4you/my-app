apiVersion: v1
kind: PersistentVolume
metadata:
  name: PV_NAME
spec:
  storageClassName: ""
  capacity:
    storage: 1Ti
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  csi:
    driver: filestore.csi.storage.gke.io
    volumeHandle: "modeInstance/FILESTORE_INSTANCE_LOCATION/FILESTORE_INSTANCE_NAME/FILESTORE_SHARE_NAME"
    volumeAttributes:
      ip: FILESTORE_INSTANCE_IP
      volume: FILESTORE_SHARE_NAME
  claimRef:
    name: PVC_NAME
    namespace: NAMESPACE
    
    
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