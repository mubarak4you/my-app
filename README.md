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
    volumeHandle: <YOUR_FILESTORE_INSTANCE_ID>/volumes/<YOUR_VOLUME_NAME>
