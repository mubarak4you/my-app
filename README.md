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
    image: nginx:stable-alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: fileserver
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
  - name: fileserver
    persistentVolumeClaim:
      claimName: fileserver
