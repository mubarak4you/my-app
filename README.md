[alabimu@10-74-129-4 ~]$ kubectl get pod sysdig-agent-dw2nf -n kube-system -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2025-04-23T19:02:49Z"
  generateName: sysdig-agent-
  labels:
    app: sysdig-agent
    app.kubernetes.io/instance: sysdig
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: agent
    app.kubernetes.io/version: 13.5.0
    controller-revision-hash: 566ffb5778
    helm.sh/chart: agent-1.30.0
    pod-template-generation: "1"
  name: sysdig-agent-dw2nf
  namespace: kube-system
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: DaemonSet
    name: sysdig-agent
    uid: 2002a97f-80ba-4ef3-9a2f-4a5ad87e36a2
  resourceVersion: "21824"
  uid: 59eee787-28f9-4e59-802b-6889ad175a5b
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchFields:
          - key: metadata.name
            operator: In
            values:
            - gke-gke-etgke-3-np-u-nap-n2-standard--fb635baf-srzp
  containers:
  - env:
    - name: K8S_NODE
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: spec.nodeName
    - name: SYSDIG_BPF_PROBE
    - name: SYSDIG_AGENT_DRIVER
      value: legacy_ebpf
    - name: http_proxy
      value: http://proxy.ebiz.verizon.com:9290
    - name: https_proxy
      value: http://proxy.ebiz.verizon.com:9290
    - name: no_proxy
      value: 172.20.0.1,10.100.0.1,127.0.0.1,localhost,.verizon.com,.vzwcorp.com,.vzbi.com,10.74.132.65,192.168.0.1,.googleapis.com,164.118.64.0/20,100.96.0.0/12,100.65.0.0/16
    image: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/sysdig/agent-slim:13.6.0
    imagePullPolicy: IfNotPresent
    name: sysdig
    readinessProbe:
      failureThreshold: 9
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 24483
        scheme: HTTP
      initialDelaySeconds: 90
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    resources:
      requests:
        cpu: 250m
        memory: 348Mi
    securityContext:
      allowPrivilegeEscalation: true
      privileged: true
      readOnlyRootFilesystem: false
      runAsNonRoot: false
      runAsUser: 0
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /host/dev
      name: dev-vol
      readOnly: true
    - mountPath: /host/usr
      name: usr-vol
      readOnly: true
    - mountPath: /host/proc
      name: proc-vol
      readOnly: true
    - mountPath: /host/run
      name: run-vol
    - mountPath: /dev/shm
      name: dshm
    - mountPath: /opt/draios/etc/kubernetes/config
      name: sysdig-agent-config
    - mountPath: /opt/draios/etc/kubernetes/secrets
      name: sysdig-agent-secrets
    - mountPath: /etc/podinfo
      name: podinfo
    - mountPath: /host/etc
      name: etc-vol
      readOnly: true
    - mountPath: /host/var/lib
      name: varlib-vol
    - mountPath: /host/var/data
      name: vardata-vol
    - mountPath: /host/var/run
      name: varrun-vol
    - mountPath: /root/.sysdig
      name: bpf-probes
    - mountPath: /sys/kernel/debug
      name: sys-tracing
      readOnly: true
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-qchcx
      readOnly: true
  dnsPolicy: ClusterFirstWithHostNet
  enableServiceLinks: true
  hostNetwork: true
  hostPID: true
  imagePullSecrets:
  - name: go0v-vzdocker
  initContainers:
  - env:
    - name: SYSDIG_BPF_PROBE
    - name: SYSDIG_AGENT_DRIVER
      value: legacy_ebpf
    - name: http_proxy
      value: http://proxy.ebiz.verizon.com:9290
    - name: https_proxy
      value: http://proxy.ebiz.verizon.com:9290
    - name: no_proxy
      value: 172.20.0.1,10.100.0.1,127.0.0.1,localhost,.verizon.com,.vzwcorp.com,.vzbi.com,10.74.132.65,192.168.0.1,.googleapis.com,164.118.64.0/20,100.96.0.0/12,100.65.0.0/16
    image: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/sysdig/agent-kmodule:13.6.0
    imagePullPolicy: IfNotPresent
    name: sysdig-agent-kmodule
    resources:
      limits:
        cpu: "1"
        memory: 512Mi
      requests:
        cpu: 250m
        memory: 348Mi
    securityContext:
      allowPrivilegeEscalation: true
      privileged: true
      readOnlyRootFilesystem: false
      runAsNonRoot: false
      runAsUser: 0
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /etc/modprobe.d
      name: modprobe-d
      readOnly: true
    - mountPath: /host/boot
      name: boot-vol
      readOnly: true
    - mountPath: /host/etc
      name: etc-vol
      readOnly: true
    - mountPath: /host/lib/modules
      name: modules-vol
      readOnly: true
    - mountPath: /host/usr
      name: usr-vol
      readOnly: true
    - mountPath: /root/.sysdig
      name: bpf-probes
    - mountPath: /sys/kernel/debug
      name: sys-tracing
      readOnly: true
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-qchcx
      readOnly: true
  nodeName: gke-gke-etgke-3-np-u-nap-n2-standard--fb635baf-srzp
  preemptionPolicy: PreemptLowerPriority
  priority: 2000000000
  priorityClassName: system-cluster-critical
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: sysdig-agent
  serviceAccountName: sysdig-agent
  terminationGracePeriodSeconds: 5
  tolerations:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
  - effect: NoSchedule
    key: node-role.kubernetes.io/control-plane
  - effect: NoSchedule
    key: node-role.kubernetes.io/controlplane
    operator: Equal
    value: "true"
  - effect: NoExecute
    key: node-role.kubernetes.io/etcd
    operator: Equal
    value: "true"
  - effect: NoExecute
    key: CriticalAddonsOnly
    operator: Equal
    value: "true"
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
  - effect: NoSchedule
    key: node.kubernetes.io/disk-pressure
    operator: Exists
  - effect: NoSchedule
    key: node.kubernetes.io/memory-pressure
    operator: Exists
  - effect: NoSchedule
    key: node.kubernetes.io/pid-pressure
    operator: Exists
  - effect: NoSchedule
    key: node.kubernetes.io/unschedulable
    operator: Exists
  - effect: NoSchedule
    key: node.kubernetes.io/network-unavailable
    operator: Exists
  volumes:
  - hostPath:
      path: /dev
      type: ""
    name: dev-vol
  - hostPath:
      path: /usr
      type: ""
    name: usr-vol
  - hostPath:
      path: /proc
      type: ""
    name: proc-vol
  - hostPath:
      path: /run
      type: ""
    name: run-vol
  - emptyDir:
      medium: Memory
    name: dshm
  - configMap:
      defaultMode: 420
      name: sysdig-agent
      optional: true
    name: sysdig-agent-config
  - name: sysdig-agent-secrets
    secret:
      defaultMode: 420
      secretName: sysdig-access-key
  - downwardAPI:
      defaultMode: 420
      items:
      - fieldRef:
          apiVersion: v1
          fieldPath: metadata.namespace
        path: namespace
      - fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
        path: name
    name: podinfo
  - hostPath:
      path: /etc/modprobe.d
      type: ""
    name: modprobe-d
  - hostPath:
      path: /boot
      type: ""
    name: boot-vol
  - hostPath:
      path: /lib/modules
      type: ""
    name: modules-vol
  - hostPath:
      path: /var/run
      type: ""
    name: varrun-vol
  - hostPath:
      path: /etc
      type: ""
    name: etc-vol
  - hostPath:
      path: /var/lib
      type: ""
    name: varlib-vol
  - hostPath:
      path: /var/data
      type: ""
    name: vardata-vol
  - emptyDir: {}
    name: bpf-probes
  - hostPath:
      path: /sys/kernel/debug
      type: ""
    name: sys-tracing
  - name: kube-api-access-qchcx
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2025-04-23T19:03:29Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2025-04-23T19:03:56Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2025-04-23T19:05:38Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2025-04-23T19:05:38Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2025-04-23T19:02:52Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://27d7283d6ec438051a3dc993a2d1520b5a7a4db0874c80b16d9cf83818d75d67
    image: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/sysdig/agent-slim:13.6.0
    imageID: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/sysdig/agent-slim@sha256:fbf1cf7985796b4903f8cab99bbb06157f75d02f83c2f812b77ebf6f0fbf2ee6
    lastState: {}
    name: sysdig
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2025-04-23T19:04:08Z"
    volumeMounts:
    - mountPath: /host/dev
      name: dev-vol
      readOnly: true
      recursiveReadOnly: Disabled
    - mountPath: /host/usr
      name: usr-vol
      readOnly: true
      recursiveReadOnly: Disabled
    - mountPath: /host/proc
      name: proc-vol
      readOnly: true
      recursiveReadOnly: Disabled
    - mountPath: /host/run
      name: run-vol
    - mountPath: /dev/shm
      name: dshm
    - mountPath: /opt/draios/etc/kubernetes/config
      name: sysdig-agent-config
    - mountPath: /opt/draios/etc/kubernetes/secrets
      name: sysdig-agent-secrets
    - mountPath: /etc/podinfo
      name: podinfo
    - mountPath: /host/etc
      name: etc-vol
      readOnly: true
      recursiveReadOnly: Disabled
    - mountPath: /host/var/lib
      name: varlib-vol
    - mountPath: /host/var/data
      name: vardata-vol
    - mountPath: /host/var/run
      name: varrun-vol
    - mountPath: /root/.sysdig
      name: bpf-probes
    - mountPath: /sys/kernel/debug
      name: sys-tracing
      readOnly: true
      recursiveReadOnly: Disabled
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-qchcx
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 164.118.64.13
  hostIPs:
  - ip: 164.118.64.13
  initContainerStatuses:
  - containerID: containerd://fba26274028dca57d5a0c05fd7d6c4ec0816a1d752841e6739ccd1de95993ee0
    image: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/sysdig/agent-kmodule:13.6.0
    imageID: go0v-vzdocker.oneartifactoryprod.verizon.com/containers/sysdig/agent-kmodule@sha256:b9262f163187e806e354bdde403a57818b7433c3d68cef03e2f918e3caa21cdb
    lastState: {}
    name: sysdig-agent-kmodule
    ready: true
    restartCount: 0
    started: false
    state:
      terminated:
        containerID: containerd://fba26274028dca57d5a0c05fd7d6c4ec0816a1d752841e6739ccd1de95993ee0
        exitCode: 0
        finishedAt: "2025-04-23T19:03:56Z"
        reason: Completed
        startedAt: "2025-04-23T19:03:29Z"
    volumeMounts:
    - mountPath: /etc/modprobe.d
      name: modprobe-d
      readOnly: true
      recursiveReadOnly: Disabled
    - mountPath: /host/boot
      name: boot-vol
      readOnly: true
      recursiveReadOnly: Disabled
    - mountPath: /host/etc
      name: etc-vol
      readOnly: true
      recursiveReadOnly: Disabled
    - mountPath: /host/lib/modules
      name: modules-vol
      readOnly: true
      recursiveReadOnly: Disabled
    - mountPath: /host/usr
      name: usr-vol
      readOnly: true
      recursiveReadOnly: Disabled
    - mountPath: /root/.sysdig
      name: bpf-probes
    - mountPath: /sys/kernel/debug
      name: sys-tracing
      readOnly: true
      recursiveReadOnly: Disabled
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-qchcx
      readOnly: true
      recursiveReadOnly: Disabled
  phase: Running
  podIP: 164.118.64.13
  podIPs:
  - ip: 164.118.64.13
  qosClass: Burstable
  startTime: "2025-04-23T19:02:52Z"