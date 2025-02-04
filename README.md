Provision Storage
Creating a PVC on a Block Volume
The CSI plugin enables dynamic provisioning of block volumes. The default storage class ("oci-bv-verizon") facilitates this process. PVC manifests that lack an explicit storageClassName will utilize "oci-bv-verizon" by default. Refer to the provided guide for detailed instructions on provisioning and mounting block volumes with PVCs.

1. Create a PVC:#
This PVC manifest specifies the storage class "oci-bv-verizon" and storage size. Use the following command to create the PVC.

kubectl apply -f https://gitlab.verizon.com/oracle-containers/oke-sample-applications/-/raw/main/samples/storage-block-volume/csi-bvs-pvc.yaml
2. Verify PVC Status:#
Use kubectl get pvc to confirm the PVC creation and its status. A status of Pending indicates the PVC is waiting for a Pod to use the volume.

3. Create a Pod with PVC:#
This Pod manifest will bind the PVC to a persistent volume and mount the volume in a Pod. Use the following command to create the Pod.

kubectl apply -f https://gitlab.verizon.com/oracle-containers/oke-sample-applications/-/raw/main/samples/storage-block-volume/pod.yaml
4. Verify PVC Binding:#
Use kubectl get pvc again to see if the PVC has bound to a persistent volume.

5. Verify Pod Usage:#
Use kubectl describe pod bv-sample to confirm the Pod utilizes the bound PVC.

6. Clean up:#
When finished, delete the Pod and PVC using kubectl delete pod bv-sample and kubectl delete pvc bv-sample.

note
Always delete application resources before deleting your OKE cluster to ensure that the CSI Volume Plugin cleans up associated OCI block volumes.








Acceptance Criteria
1. Add page titled "Use Persistent Disks" to the "Provision Storage" folder in the GKE > Guides section.

2. The "Use Persistent Disks" page describes the following:

Describe which KMS encrypted StorageClasses are available for use.
Provide an example that creates a PersistentVolumeClaim and creates a Pod that consumes the volume



[alabimu@10-74-128-76 ~]$ ls
sample-pod.yaml  sample-pvc.yaml
[alabimu@10-74-128-76 ~]$ cat *
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
      #image: gpool-zdocker.oneartifactoryprod.verizon.com/containers/cicd/busybox:1.35.0
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
