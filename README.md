apiVersion: apps/v1
kind: Deployment
metadata:
  name: reader
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reader
  template:
    metadata:
      labels:
        app: reader
    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: fileserver
          mountPath: /usr/share/nginx/html # the shared directory 
          readOnly: true
      volumes:
      - name: fileserver
        persistentVolumeClaim:
          claimName: fileserver
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