apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: bv-sample
  namespace: strg
spec:
  storageClassName: "oci-bv-verizon"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: bv-sample
  namespace: strg
spec:
  volumes:
    - name: bv-sample
      persistentVolumeClaim:
        claimName: bv-sample
  containers:
    - name: app
      image: goovl-zdocker.oneartifactoryprod.verizon.com/containers/cicd/busybox:1.35.0
      command: ["sleep", "infinity"]
      volumeMounts:
        - mountPath: /tmp
          name: bv-sample
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
  securityContext:
    fsGroup: 1001
  supplementalGroups: [1001]
