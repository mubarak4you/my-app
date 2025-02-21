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

```bash
kubectl apply -f https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/filestore/sample-pv.yaml
```

The `volumeHandle` uniquely identifies the Filestore instance in GCP, specifying the storage location and volume name. The `volumeAttributes` contain the IP address of the Filestore instance, which is required for connecting to it.


#### 2. Create a PersistentVolumeClaim (PVC)
This PVC manifest specifies the `csi-gce-filestore-cmek` storage class and requests 50Gi of storage.

```bash
kubectl apply -f https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/filestore/sample-pvc.yaml
```

#### 3. Create a Pod with PVC
This Pod manifest binds the PVC to a persistent volume and mounts it in a container.

```bash
kubectl apply -f https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/filestore/sample-pod.yaml
```

#### 4. Verify PVC Binding
Use `kubectl get pvc` to check if the PVC has been bound to a persistent volume.

#### 5. Verify Pod Usage
Use `kubectl describe pod busybox` to describe the Pod to confirm it is using the bound PVC.

#### 6. Write to Filestore
To verify that the Filestore instance is accessible and writable, exec into the container and create a test file:
```bash
kubectl exec -it busybox -- touch /usr/share/nginx/html/testfile.txt
```

#### 7. Clean Up
When finished, delete the Pod and PVC:
```bash
kubectl delete pod busybox
kubectl delete pvc fileserver-pvc
```

:::note
Always delete application resources before deleting your GKE cluster to ensure that the CSI Volume Plugin cleans up associated persistent disks properly.
:::










#### 6. Write to Filestore
To verify that the Filestore instance is accessible and writable, exec into the container and create a test file:
```bash
kubectl exec -it busybox -- touch /usr/share/nginx/html/testfile.txt
```

i want to usee this type of exapmle below instead for step 6

This demonstrates the read/write ability:
$ k exec -it busybox -- sh
/ $ ls -al /tmp/
total 4
drwxrwsr-x    2 root     1000             0 Feb 14 16:31 .
drwxr-xr-x    1 root     root          4096 Feb 17 17:27 ..
/ $ echo "Hello world!" > /tmp/hello.txt
/ $ ls -al /tmp/
total 8
drwxrwsr-x    2 root     1000             0 Feb 17 17:31 .
drwxr-xr-x    1 root     root          4096 Feb 17 17:27 ..
-rw-r--r--    1 1000     1000            13 Feb 17 17:31 hello.txt
/ $ cat /tmp/hello.txt 
Hello world!