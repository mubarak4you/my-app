# Use Persistent Disks

## Creating a PVC on a Persistent Disk

The CSI plugin enables dynamic provisioning of persistent disks in GKE. The available KMS-encrypted storage classes include:

- **Balanced Persistent Disk (`csi-gce-balanced-cmek`)**: Balanced performance and cost, backed by SSD.
- **Performance Persistent Disk (`csi-gce-ssd-cmek`)**: High performance for sensitive workloads, backed by SSD.
- **Standard Persistent Disk (`csi-gce-pd-cmek`)**: Cost-effective, backed by HDD.
- **Filestore Storage (`csi-gce-filestore-cmek`)**: NFS access for applications requiring multiple reads and writes.

PersistentVolumeClaim (PVC) manifests that specify one of these storage classes will dynamically provision storage for Pods.

### 1. Create a PVC
This PVC manifest specifies the `csi-gce-balanced-cmek` storage class and requests 50Gi of storage.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-sample
  namespace: storage
spec:
  storageClassName: "csi-gce-balanced-cmek"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
```

Apply the PVC with:
```sh
kubectl apply -f sample-pvc.yaml
```

### 2. Verify PVC Status
Check the PVC status:
```sh
kubectl get pvc -n storage
```
A status of `Pending` indicates the PVC is waiting for a Pod to use the volume.

### 3. Create a Pod with PVC
This Pod manifest binds the PVC to a persistent volume and mounts it in a container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-sample
  namespace: storage
spec:
  volumes:
    - name: pv-sample
      persistentVolumeClaim:
        claimName: pv-sample
  containers:
    - name: app
      image: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/apache:2.4.58
      command: ["sleep", "infinity"]
      volumeMounts:
        - mountPath: /tmp
          name: pv-sample
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
  securityContext:
    fsGroup: 1001
    supplementalGroups: [1001]
```

Apply the Pod with:
```sh
kubectl apply -f sample-pod.yaml
```

### 4. Verify PVC Binding
Check if the PVC has been bound to a persistent volume:
```sh
kubectl get pvc -n storage
```

### 5. Verify Pod Usage
Describe the Pod to confirm it is using the bound PVC:
```sh
kubectl describe pod pv-sample -n storage
```

### 6. Clean Up
When finished, delete the Pod and PVC:
```sh
kubectl delete pod pv-sample -n storage
kubectl delete pvc pv-sample -n storage
```

### Note
Always delete application resources before deleting your GKE cluster to ensure that the CSI Volume Plugin cleans up associated persistent disks properly.

