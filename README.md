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
