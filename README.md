---
id: filestore
title: Use Filestore
---

The guide describes the typical process for using a Kubernetes volume backed by a CSI driver in GKE.

#### Storage Classes
You must use the KMS-encrypted storage class listed below. PersistentVolumeClaim (PVC) manifests that specify this storage class will dynamically provision storage for Pods.

- **Filestore Storage (`csi-gce-filestore-cmek`)**: NFS access for applications requiring multiple reads and writes.

#### 1. Create a PersistentVolume (PV)
This PersistentVolume (PV) resource represents a pre-existing Filestore instance and is required before a PersistentVolumeClaim (PVC) can be created.

The `volumeHandle` uniquely identifies the Filestore instance in GCP, specifying the storage location and volume name. The `volumeAttributes` contain the IP address of the Filestore instance, which is required for connecting to it.

```yaml
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
```

#### 2. Create a PersistentVolumeClaim (PVC)
This PVC manifest specifies the `csi-gce-filestore-cmek` storage class and requests 10Gi of storage.

```yaml
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
```

Apply the PVC manifest:
```bash
kubectl apply -f <path-to-pvc-file>
```

#### 3. Verify PVC Status
Use `kubectl get pvc` to check the status of the PVC.

A status of `Pending` indicates the PVC is waiting for a Pod to use the volume.

#### 4. Create a Pod with PVC
This Pod manifest binds the PVC to a persistent volume and mounts it in a container.

```yaml
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
```

Apply the Pod manifest:
```bash
kubectl apply -f <path-to-pod-file>
```

#### 5. Verify PVC Binding
Use `kubectl get pvc` to check if the PVC has been bound to a persistent volume.

#### 6. Verify Pod Usage
Use `kubectl describe pod reader` to describe the Pod to confirm it is using the bound PVC.

#### 7. Write to Filestore
To verify that the Filestore instance is accessible and writable, exec into the container and create a test file:
```bash
kubectl exec -it reader -- touch /usr/share/nginx/html/testfile.txt
```

#### 8. Clean Up
When finished, delete the Pod and PVC:
```bash
kubectl delete pod reader
kubectl delete pvc fileserver-pvc
```

:::note
Always delete application resources before deleting your GKE cluster to ensure that the CSI Volume Plugin cleans up associated persistent disks properly.
:::

