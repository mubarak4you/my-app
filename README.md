apiVersion: v1
kind: Pod
metadata:
  name: pv-sample
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
