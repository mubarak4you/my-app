apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "istio-ingressgateway.fullname" . }}
  labels:
    {{- include "istio-ingressgateway.labels" . | nindent 4 }}
spec:
  selector:
    matchLabels:
      {{- include "istio-ingressgateway.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        inject.istio.io/templates: gateway
      labels:
        {{- include "istio-ingressgateway.selectorLabels" . | nindent 8 }}
    spec:
      securityContext:
        sysctls:
          - name: net.ipv4.ip_unprivileged_port_start
            value: "0"
      containers:
        - name: istio-proxy
          image: {{ include "istio-ingressgateway.image" . }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            privileged: false
            readOnlyRootFilesystem: true
            runAsGroup: 1337
            runAsNonRoot: true
            runAsUser: 1337
      imagePullSecrets:
        - name: {{ .Values.imagePullSecret }}
      initContainers:
        - image: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/kubectl:1.30.5
          name: init
          command:
            - "/bin/bash"
          args:
            - "-c"
            - >
              export PROXY_IMAGE=$(kubectl -n $POD_NAMESPACE get pod $POD_NAME -o=jsonpath='{.spec.containers[0].image}');
              if [[ $PROXY_IMAGE == "auto" ]]; then
                sleep 30;
                kubectl -n $POD_NAMESPACE delete pod $POD_NAME;
              else
                echo "ASM is ready.";
              fi;
          securityContext:
            privileged: false
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsNonRoot: true
            capabilities:
              drop: ["ALL"]
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
      serviceAccountName: {{ include "istio-ingressgateway.serviceAccountName" . }}
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "istio-ingressgateway.fullname" . }}
  labels:
    {{- include "istio-ingressgateway.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "istio-ingressgateway.fullname" . }}
  minReplicas: 3
  maxReplicas: 8
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 80
---
apiVersion: {{ include "istio-ingressgateway.pdb.apiVersion" . }}
kind: PodDisruptionBudget
metadata:
  name: {{ include "istio-ingressgateway.fullname" . }}
  labels:
    {{- include "istio-ingressgateway.labels" . | nindent 4 }}
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      {{- include "istio-ingressgateway.selectorLabels" . | nindent 6 }}
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: {{ include "istio-ingressgateway.fullname" . }}
spec:
  rules:
    - apiGroups: [""]
      resources: ["secrets"]
      verbs: ["get", "watch", "list"]
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: {{ include "istio-ingressgateway.fullname" . }}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: {{ include "istio-ingressgateway.fullname" . }}
subjects:
  - kind: ServiceAccount
    name: {{ include "istio-ingressgateway.serviceAccountName" . }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ include "istio-ingressgateway.fullname" . }}
  labels:
    {{- include "istio-ingressgateway.labels" . | nindent 4 }}
  annotations:
    networking.gke.io/load-balancer-type: "internal"
spec:
  ports:
    - name: status-port
      port: 15021
      protocol: TCP
      targetPort: 15021
    - name: https
      port: 443
  selector:
    {{- include "istio-ingressgateway.selectorLabels" . | nindent 4 }}
  type: LoadBalancer
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "istio-ingressgateway.serviceAccountName" . }}





resources:
  limits:
    cpu: 2000m
    memory: 1024Mi
  requests:
    cpu: 100m
    memory: 128Mi

serviceAccount:
  create: true
  name: istio-ingressgateway

image:
  repository: docker.io/istio/proxyv2
  tag: "1.20.2"

imagePullSecret: go0v-vzdocker






