---
id: storage
title: Use Persistent Disks
---

The guide describes the typical process for using a Kubernetes volume backed by a CSI driver in GKE.

#### Storage Classes
You must use one of the KMS-encrypted storage classes listed below. PersistentVolumeClaim (PVC) manifests that specify one of these storage classes will dynamically provision storage for Pods.


- **Balanced Persistent Disk (`csi-gce-balanced-cmek`)**: Balanced performance and cost, backed by SSD.
- **Performance Persistent Disk (`csi-gce-ssd-cmek`)**: High performance for sensitive workloads, backed by SSD.
- **Standard Persistent Disk (`csi-gce-pd-cmek`)**: Cost-effective, backed by HDD.
- **Filestore Storage (`csi-gce-filestore-cmek`)**: NFS access for applications requiring multiple reads and writes.

#### 1. Create a PVC
This [PVC manifest] specifies the `csi-gce-balanced-cmek` storage class and requests 50Gi of storage.

```bash
kubectl apply -f https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/persistent-disks/sample-pvc.yaml
```

#### 2. Verify PVC Status
Use `kubectl get pvc` to check the the status of the PVC.

A status of `Pending` indicates the PVC is waiting for a Pod to use the volume.

#### 3. Create a Pod with PVC
This [Pod manifest] binds the PVC to a persistent volume and mounts it in a container.

```bash
kubectl apply -f https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/persistent-disks/sample-pod.yaml
```

#### 4. Verify PVC Binding
Use `kubectl get pvc` to check if the PVC has been bound to a persistent volume.


#### 5. Verify Pod Usage
Use `kubectl describe pod pv-sample` to describe the Pod to confirm it is using the bound PVC.


#### 6. Clean Up
When finished, delete the Pod and PVC:
`kubectl delete pod pv-sample`
and
`kubectl delete pvc pv-sample`


:::note
Always delete application resources before deleting your GKE cluster to ensure that the CSI Volume Plugin cleans up associated persistent disks properly.
:::


[PVC manifest]: https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/persistent-disks/sample-pvc.yaml
[Pod manifest]: https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/persistent-disks/sample-pod.yaml



Using the document above the (Use Persistent Disks) as a template, create a doc like above for this ticket requirement below.


Add Filestore Storage Guide
Acceptance Criteria
1. Add page titled "Use Filestore" to the "Provision Storage" folder in the GKE > Guides section.

2. The "Use Filestore" page describes the following:

Describe which KMS encrypted StorageClass is available for use.
Provide an example of how to access a pre-existing Filestore instance. This example creates PersistentVolumeClaim and creates a Pod that consumes the volume. More details available in the Google documentation.
Verify that we can write to the filestore




I have the URL for a sample pv, pvc and pod that will go in the filestorage Guide.

This is the sample pv resource below, 
Make sure to add in the doc guide and explain 
    volumeHandle: "modeInstance/us-east4/gke-sample/vol1"
    volumeAttributes:
      ip: 192.168.224.66


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
    volumeHandle: "modeInstance/us-east4/gke-sample/vol1"
    volumeAttributes:
      ip: 192.168.224.66
      volume: vol1
  claimRef:
    namespace: default
    name: fileserver-pvc
    
    
This is the pod resource
apiVersion: v1
kind: Pod
metadata:
  name: reader
spec:
  securityContext:
    fsGroup: 1001
    supplementalGroups: [1001]
  containers:
  - name: nginx
    image: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/apache:2.4.58
    ports:
    - containerPort: 80
    volumeMounts:
    - name: fileserver-volume
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
  - name: fileserver-volume
    persistentVolumeClaim:
      claimName: fileserver-pvc


This is the pvc resource
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: fileserver-pvc
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: csi-gce-filestore-cmek
  resources:
    requests:
      storage: 10Gi


Tepmrorayly add the yaml reources until I replace them with just the link like for the sample gke application link.

