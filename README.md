[alabimu@10-74-128-76 filestore]$ cat *
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
    #volumeHandle: "modeInstance/FILESTORE_INSTANCE_LOCATION/FILESTORE_INSTANCE_NAME/FILESTORE_SHARE_NAME"
    volumeHandle: "modeInstance/us-east4/gke-sampl/vol1"
    #volumeHandle: "modeInstance/projects/vz-it-np-exhv-sharedvpc-228116/locations/us-east4/instances/gke-sample/volumes/vol1"
    #volumeHandle: "projects/vz-it-np-go0v-dev-gketst-0/locations/us-east4/instances/gke-sample"
    volumeAttributes:
      ip: 192.168.224.66
      volume: vol1
  claimRef:
    namespace: default
    name: fileserver-pvc

[alabimu@10-74-128-76 filestore]$ 
[alabimu@10-74-128-76 filestore]$ 
[alabimu@10-74-128-76 filestore]$ kubectl get pods
NAME     READY   STATUS     RESTARTS   AGE
reader   0/2     Init:0/1   0          8m42s
[alabimu@10-74-128-76 filestore]$ kubectl get pvc -n default
NAME                    STATUS        VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS                  VOLUMEATTRIBUTESCLASS   AGE
fileserver-pvc          Bound         fileserver-pv   10Gi       RWX            csi-gce-filestore-cmek        <unset>                 8m47s
test-store-test-set-0   Terminating                                             csi-gce-filestore-test-cmek   <unset>                 357d
[alabimu@10-74-128-76 filestore]$ kubectl get pv
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS             VOLUMEATTRIBUTESCLASS   REASON   AGE
fileserver-pv   10Gi       RWX            Retain           Bound    default/fileserver-pvc   csi-gce-filestore-cmek   <unset>                          8m51s
[alabimu@10-74-128-76 filestore]$ kubectl describe pod reader
Name:             reader
Namespace:        default
Priority:         0
Service Account:  default
Node:             gke-gke-core-np-us-e-nap-e2-standard--9fa92533-1ak1/164.118.64.114
Start Time:       Fri, 07 Feb 2025 11:36:42 -0500
Labels:           security.istio.io/tlsMode=istio
                  service.istio.io/canonical-name=reader
                  service.istio.io/canonical-revision=latest
Annotations:      cloud.google.com/cluster_autoscaler_unhelpable_since: 2025-02-07T16:36:41+0000
                  cloud.google.com/cluster_autoscaler_unhelpable_until: Inf
                  istio.io/rev: asm-managed-stable
                  k8s.v1.cni.cncf.io/networks: default/istio-cni
                  kubectl.kubernetes.io/default-container: nginx
                  kubectl.kubernetes.io/default-logs-container: nginx
                  prometheus.io/path: /stats/prometheus
                  prometheus.io/port: 15020
                  prometheus.io/scrape: true
                  sidecar.istio.io/interceptionMode: REDIRECT
                  sidecar.istio.io/status:
                    {"initContainers":["istio-validation"],"containers":["istio-proxy"],"volumes":["workload-socket","credential-socket","workload-certs","ist...
                  traffic.sidecar.istio.io/excludeInboundPorts: 15020
                  traffic.sidecar.istio.io/includeInboundPorts: *
                  traffic.sidecar.istio.io/includeOutboundIPRanges: *
Status:           Pending
IP:               
IPs:              <none>
Init Containers:
  istio-validation:
    Container ID:  
    Image:         gcr.io/gke-release/asm/proxyv2:1.19.10-asm.24
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Args:
      istio-iptables
      -p
      15001
      -z
      15006
      -u
      1337
      -m
      REDIRECT
      -i
      *
      -x
      
      -b
      *
      -d
      15090,15021,15020
      --log_output_level=default:info
      --run-validation
      --skip-rule-apply
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Limits:
      cpu:     2
      memory:  1Gi
    Requests:
      cpu:     100m
      memory:  128Mi
    Environment:
      CA_PROVIDER:             GoogleCA
      CA_ROOT_CA:              /etc/ssl/certs/ca-certificates.crt
      CA_TRUSTANCHOR:          
      FLEET_PROJECT_NUMBER:    669922244422
      GCP_METADATA:            vz-it-np-go0v-dev-gketst-0|669922244422|gke-core-np-us-east4-go0v|us-east4
      OUTPUT_CERTS:            /etc/istio/proxy
      PROXY_CONFIG_XDS_AGENT:  true
      XDS_AUTH_PROVIDER:       gcp
      XDS_ROOT_CA:             /etc/ssl/certs/ca-certificates.crt
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l2nmw (ro)
Containers:
  nginx:
    Container ID:    
    Image:           go0v-vzdocker.oneartifactoryprod.verizon.com/containers/cicd/apache:2.4.58
    Image ID:        
    Port:            80/TCP
    Host Port:       0/TCP
    SeccompProfile:  RuntimeDefault
    State:           Waiting
      Reason:        PodInitializing
    Ready:           False
    Restart Count:   0
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        100m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /usr/share/nginx/html from fileserver-volume (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l2nmw (ro)
  istio-proxy:
    Container ID:  
    Image:         gcr.io/gke-release/asm/proxyv2:1.19.10-asm.24
    Image ID:      
    Port:          15090/TCP
    Host Port:     0/TCP
    Args:
      proxy
      sidecar
      --domain
      $(POD_NAMESPACE).svc.cluster.local
      --proxyLogLevel=warning
      --proxyComponentLogLevel=misc:error
      --log_output_level=default:info
      --stsPort=15463
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Limits:
      cpu:     2
      memory:  1Gi
    Requests:
      cpu:      100m
      memory:   128Mi
    Readiness:  http-get http://:15021/healthz/ready delay=1s timeout=3s period=2s #success=1 #failure=30
    Environment:
      JWT_POLICY:                    third-party-jwt
      PILOT_CERT_PROVIDER:           system
      CA_ADDR:                       meshca.googleapis.com:443
      POD_NAME:                      reader (v1:metadata.name)
      POD_NAMESPACE:                 default (v1:metadata.namespace)
      INSTANCE_IP:                    (v1:status.podIP)
      SERVICE_ACCOUNT:                (v1:spec.serviceAccountName)
      HOST_IP:                        (v1:status.hostIP)
      ISTIO_CPU_LIMIT:               2 (limits.cpu)
      PROXY_CONFIG:                  {"discoveryAddress":"meshconfig.googleapis.com:443","tracing":{"stackdriver":{}},"proxyMetadata":{"CA_PROVIDER":"GoogleCA","CA_ROOT_CA":"/etc/ssl/certs/ca-certificates.crt","CA_TRUSTANCHOR":"","FLEET_PROJECT_NUMBER":"669922244422","GCP_METADATA":"vz-it-np-go0v-dev-gketst-0|669922244422|gke-core-np-us-east4-go0v|us-east4","OUTPUT_CERTS":"/etc/istio/proxy","PROXY_CONFIG_XDS_AGENT":"true","XDS_AUTH_PROVIDER":"gcp","XDS_ROOT_CA":"/etc/ssl/certs/ca-certificates.crt"},"meshId":"proj-669922244422"}
                                     
      ISTIO_META_POD_PORTS:          [
                                         {"containerPort":80,"protocol":"TCP"}
                                     ]
      ISTIO_META_APP_CONTAINERS:     nginx
      GOMEMLIMIT:                    1073741824 (limits.memory)
      GOMAXPROCS:                    2 (limits.cpu)
      ISTIO_META_NODE_NAME:           (v1:spec.nodeName)
      ISTIO_META_INTERCEPTION_MODE:  REDIRECT
      ISTIO_META_WORKLOAD_NAME:      reader
      ISTIO_META_OWNER:              kubernetes://apis/v1/namespaces/default/pods/reader
      ISTIO_META_MESH_ID:            proj-669922244422
      TRUST_DOMAIN:                  vz-it-np-go0v-dev-gketst-0.svc.id.goog
      CA_PROVIDER:                   GoogleCA
      CA_ROOT_CA:                    /etc/ssl/certs/ca-certificates.crt
      CA_TRUSTANCHOR:                
      FLEET_PROJECT_NUMBER:          669922244422
      GCP_METADATA:                  vz-it-np-go0v-dev-gketst-0|669922244422|gke-core-np-us-east4-go0v|us-east4
      OUTPUT_CERTS:                  /etc/istio/proxy
      PROXY_CONFIG_XDS_AGENT:        true
      XDS_AUTH_PROVIDER:             gcp
      XDS_ROOT_CA:                   /etc/ssl/certs/ca-certificates.crt
      ISTIO_DELTA_XDS:               false
      ISTIO_META_CLUSTER_ID:         cn-vz-it-np-go0v-dev-gketst-0-us-east4-gke-core-np-us-east4-go0v
      ISTIO_META_CONTROL_PLANE:      projects/vz-it-np-go0v-dev-gketst-0/locations/us-east4/clusters/gke-core-np-us-east4-go0v/controlPlanes/asm-managed-stable
    Mounts:
      /etc/istio/pod from istio-podinfo (rw)
      /etc/istio/proxy from istio-envoy (rw)
      /var/lib/istio/data from istio-data (rw)
      /var/run/secrets/credential-uds from credential-socket (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l2nmw (ro)
      /var/run/secrets/tokens from istio-token (rw)
      /var/run/secrets/workload-spiffe-credentials from workload-certs (rw)
      /var/run/secrets/workload-spiffe-uds from workload-socket (rw)
Conditions:
  Type                        Status
  PodReadyToStartContainers   False 
  Initialized                 False 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  workload-socket:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  credential-socket:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  workload-certs:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  istio-envoy:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     Memory
    SizeLimit:  <unset>
  istio-data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  istio-podinfo:
    Type:  DownwardAPI (a volume populated by information about the pod)
    Items:
      metadata.labels -> labels
      metadata.annotations -> annotations
  istio-token:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  43200
  fileserver-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  fileserver-pvc
    ReadOnly:   false
  kube-api-access-l2nmw:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason             Age                    From                                   Message
  ----     ------             ----                   ----                                   -------
  Warning  FailedScheduling   8m54s                  gke.io/optimize-utilization-scheduler  0/2 nodes are available: persistentvolumeclaim "fileserver-pvc" not found. preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.
  Normal   NotTriggerScaleUp  8m54s                  cluster-autoscaler                     pod didn't trigger scale-up (it wouldn't fit if a new node is added): 3 persistentvolumeclaim "fileserver-pvc" not found
  Normal   Scheduled          8m53s                  gke.io/optimize-utilization-scheduler  Successfully assigned default/reader to gke-gke-core-np-us-e-nap-e2-standard--9fa92533-1ak1
  Warning  FailedMount        6m21s (x6 over 6m52s)  kubelet                                MountVolume.MountDevice failed for volume "fileserver-pv" : rpc error: code = Aborted desc = An operation with the given volume key modeInstance/us-east4/gke-sampl/vol1 already exists.
 --- Most likely a long process is still running to completion. Retrying.
  Warning  FailedMount  44s (x3 over 6m52s)  kubelet  MountVolume.MountDevice failed for volume "fileserver-pv" : rpc error: code = DeadlineExceeded desc = context deadline exceeded