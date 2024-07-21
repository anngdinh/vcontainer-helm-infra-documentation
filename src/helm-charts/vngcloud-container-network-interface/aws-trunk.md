# Trunk config

## kubectl get pod -A -owide

```bash
NAMESPACE      NAME                             READY   STATUS    RESTARTS   AGE     IP              NODE                                              NOMINATED NODE   READINESS GATES
default        nsenter-n2qvfn                   1/1     Running   0          5m17s   172.31.0.225    ip-172-31-0-225.ap-southeast-2.compute.internal   <none>           <none>
kube-system    aws-node-mbmjp                   2/2     Running   0          11m     172.31.0.225    ip-172-31-0-225.ap-southeast-2.compute.internal   <none>           <none>
kube-system    coredns-585d4c556b-z6ddv         1/1     Running   0          18m     172.31.12.149   ip-172-31-0-225.ap-southeast-2.compute.internal   <none>           <none>
kube-system    coredns-585d4c556b-z6qw6         1/1     Running   0          18m     172.31.3.129    ip-172-31-0-225.ap-southeast-2.compute.internal   <none>           <none>
kube-system    kube-proxy-szr2x                 1/1     Running   0          15m     172.31.0.225    ip-172-31-0-225.ap-southeast-2.compute.internal   <none>           <none>
my-namespace   my-deployment-758d6cbf7c-k6nvg   1/1     Running   0          5m39s   172.31.2.37     ip-172-31-0-225.ap-southeast-2.compute.internal   <none>           <none>
my-namespace   my-deployment-758d6cbf7c-n6b5v   1/1     Running   0          5m39s   172.31.9.169    ip-172-31-0-225.ap-southeast-2.compute.internal   <none>           <none>
```

## kubectl -n kube-system get pod aws-node-mbmjp -oyaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-07-18T02:41:59Z"
  generateName: aws-node-
  labels:
    app.kubernetes.io/instance: aws-vpc-cni
    app.kubernetes.io/name: aws-node
    controller-revision-hash: 6869bbb74c
    k8s-app: aws-node
    pod-template-generation: "2"
  name: aws-node-mbmjp
  namespace: kube-system
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: DaemonSet
    name: aws-node
    uid: 899c79fb-a5fc-4ef9-bb5c-23f06d466a01
  resourceVersion: "1955"
  uid: e3776cf9-85ce-40f3-8af5-7d7c95287294
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchFields:
          - key: metadata.name
            operator: In
            values:
            - ip-172-31-0-225.ap-southeast-2.compute.internal
  containers:
  - env:
    - name: ADDITIONAL_ENI_TAGS
      value: '{}'
    - name: ANNOTATE_POD_IP
      value: "false"
    - name: AWS_VPC_CNI_NODE_PORT_SUPPORT
      value: "true"
    - name: AWS_VPC_ENI_MTU
      value: "9001"
    - name: AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG
      value: "false"
    - name: AWS_VPC_K8S_CNI_EXTERNALSNAT
      value: "false"
    - name: AWS_VPC_K8S_CNI_LOGLEVEL
      value: DEBUG
    - name: AWS_VPC_K8S_CNI_LOG_FILE
      value: /host/var/log/aws-routed-eni/ipamd.log
    - name: AWS_VPC_K8S_CNI_RANDOMIZESNAT
      value: prng
    - name: AWS_VPC_K8S_CNI_VETHPREFIX
      value: eni
    - name: AWS_VPC_K8S_PLUGIN_LOG_FILE
      value: /var/log/aws-routed-eni/plugin.log
    - name: AWS_VPC_K8S_PLUGIN_LOG_LEVEL
      value: DEBUG
    - name: CLUSTER_NAME
      value: annd2-mAE2VxaY
    - name: DISABLE_INTROSPECTION
      value: "false"
    - name: DISABLE_METRICS
      value: "false"
    - name: DISABLE_NETWORK_RESOURCE_PROVISIONING
      value: "false"
    - name: ENABLE_IPv4
      value: "true"
    - name: ENABLE_IPv6
      value: "false"
    - name: ENABLE_POD_ENI
      value: "true"
    - name: ENABLE_PREFIX_DELEGATION
      value: "false"
    - name: ENABLE_SUBNET_DISCOVERY
      value: "true"
    - name: NETWORK_POLICY_ENFORCING_MODE
      value: standard
    - name: VPC_CNI_VERSION
      value: v1.18.1
    - name: VPC_ID
      value: vpc-09d4ca72082a8bb94
    - name: WARM_ENI_TARGET
      value: "1"
    - name: WARM_PREFIX_TARGET
      value: "1"
    - name: MY_NODE_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: spec.nodeName
    - name: MY_POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    image: 602401143452.dkr.ecr.ap-southeast-2.amazonaws.com/amazon-k8s-cni:v1.18.1-eksbuild.3
    imagePullPolicy: IfNotPresent
    livenessProbe:
      exec:
        command:
        - /app/grpc-health-probe
        - -addr=:50051
        - -connect-timeout=5s
        - -rpc-timeout=5s
      failureThreshold: 3
      initialDelaySeconds: 60
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 10
    name: aws-node
    ports:
    - containerPort: 61678
      hostPort: 61678
      name: metrics
      protocol: TCP
    readinessProbe:
      exec:
        command:
        - /app/grpc-health-probe
        - -addr=:50051
        - -connect-timeout=5s
        - -rpc-timeout=5s
      failureThreshold: 3
      initialDelaySeconds: 1
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 10
    resources:
      requests:
        cpu: 25m
    securityContext:
      capabilities:
        add:
        - NET_ADMIN
        - NET_RAW
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /host/opt/cni/bin
      name: cni-bin-dir
    - mountPath: /host/etc/cni/net.d
      name: cni-net-dir
    - mountPath: /host/var/log/aws-routed-eni
      name: log-dir
    - mountPath: /var/run/aws-node
      name: run-dir
    - mountPath: /run/xtables.lock
      name: xtables-lock
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-n8nfv
      readOnly: true
  - args:
    - --enable-ipv6=false
    - --enable-network-policy=false
    - --enable-cloudwatch-logs=false
    - --enable-policy-event-logs=false
    - --metrics-bind-addr=:8162
    - --health-probe-bind-addr=:8163
    - --conntrack-cache-cleanup-period=300
    env:
    - name: MY_NODE_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: spec.nodeName
    - name: ENABLE_POD_ENI
      value: "true"
    image: 602401143452.dkr.ecr.ap-southeast-2.amazonaws.com/amazon/aws-network-policy-agent:v1.1.1-eksbuild.2
    imagePullPolicy: IfNotPresent
    name: aws-eks-nodeagent
    resources:
      requests:
        cpu: 25m
    securityContext:
      capabilities:
        add:
        - NET_ADMIN
      privileged: true
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /host/opt/cni/bin
      name: cni-bin-dir
    - mountPath: /sys/fs/bpf
      name: bpf-pin-path
    - mountPath: /var/log/aws-routed-eni
      name: log-dir
    - mountPath: /var/run/aws-node
      name: run-dir
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-n8nfv
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  hostNetwork: true
  initContainers:
  - env:
    - name: DISABLE_TCP_EARLY_DEMUX
      value: "false"
    - name: ENABLE_IPv6
      value: "false"
    - name: ENABLE_POD_ENI
      value: "true"
    image: 602401143452.dkr.ecr.ap-southeast-2.amazonaws.com/amazon-k8s-cni-init:v1.18.1-eksbuild.3
    imagePullPolicy: IfNotPresent
    name: aws-vpc-cni-init
    resources:
      requests:
        cpu: 25m
    securityContext:
      privileged: true
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /host/opt/cni/bin
      name: cni-bin-dir
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-n8nfv
      readOnly: true
  nodeName: ip-172-31-0-225.ap-southeast-2.compute.internal
  preemptionPolicy: PreemptLowerPriority
  priority: 2000001000
  priorityClassName: system-node-critical
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: aws-node
  serviceAccountName: aws-node
  terminationGracePeriodSeconds: 10
  tolerations:
  - operator: Exists
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
      path: /sys/fs/bpf
      type: ""
    name: bpf-pin-path
  - hostPath:
      path: /opt/cni/bin
      type: ""
    name: cni-bin-dir
  - hostPath:
      path: /etc/cni/net.d
      type: ""
    name: cni-net-dir
  - hostPath:
      path: /var/log/aws-routed-eni
      type: DirectoryOrCreate
    name: log-dir
  - hostPath:
      path: /var/run/aws-node
      type: DirectoryOrCreate
    name: run-dir
  - hostPath:
      path: /run/xtables.lock
      type: ""
    name: xtables-lock
  - name: kube-api-access-n8nfv
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
    lastTransitionTime: "2024-07-18T02:42:00Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2024-07-18T02:42:00Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2024-07-18T02:42:02Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2024-07-18T02:42:02Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2024-07-18T02:41:59Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://ed02fdcedf3d8e87a951bdd1027e531a835ca2f574ec156d2f7f98ca5f74fda0
    image: 602401143452.dkr.ecr.ap-southeast-2.amazonaws.com/amazon/aws-network-policy-agent:v1.1.1-eksbuild.2
    imageID: 602401143452.dkr.ecr.ap-southeast-2.amazonaws.com/amazon/aws-network-policy-agent@sha256:82a3289a0551940323057e2fb162f256dd5fd41f8945f6ad6f9a2e4c24648a65
    lastState: {}
    name: aws-eks-nodeagent
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2024-07-18T02:42:01Z"
  - containerID: containerd://7b3b0a9ec4180b356174fabc89c4d807824e9e8f2bd559b0bef71f9e656e26de
    image: 066635153087.dkr.ecr.il-central-1.amazonaws.com/amazon-k8s-cni:v1.18.1-eksbuild.3
    imageID: sha256:86800e25303d102ce2d081833ccb7b51b354d610faf3198980458f779590d6b8
    lastState: {}
    name: aws-node
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2024-07-18T02:42:00Z"
  hostIP: 172.31.0.225
  hostIPs:
  - ip: 172.31.0.225
  initContainerStatuses:
  - containerID: containerd://8a47ad5f432f153ad598facbe6c7cdce78f78c13f99863b67600460bb4197953
    image: 066635153087.dkr.ecr.il-central-1.amazonaws.com/amazon-k8s-cni-init:v1.18.1-eksbuild.3
    imageID: sha256:3f0789d4b13f6149abed0acb024fc383f0569f98e1d54316410b849bdd3fe71e
    lastState: {}
    name: aws-vpc-cni-init
    ready: true
    restartCount: 0
    started: false
    state:
      terminated:
        containerID: containerd://8a47ad5f432f153ad598facbe6c7cdce78f78c13f99863b67600460bb4197953
        exitCode: 0
        finishedAt: "2024-07-18T02:42:00Z"
        reason: Completed
        startedAt: "2024-07-18T02:41:59Z"
  phase: Running
  podIP: 172.31.0.225
  podIPs:
  - ip: 172.31.0.225
  qosClass: Burstable
  startTime: "2024-07-18T02:41:59Z"
```

## ip a

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 02:f0:a2:fb:16:95 brd ff:ff:ff:ff:ff:ff
    inet 172.31.0.225/20 brd 172.31.15.255 scope global dynamic eth0
       valid_lft 2981sec preferred_lft 2981sec
    inet6 fe80::f0:a2ff:fefb:1695/64 scope link 
       valid_lft forever preferred_lft forever
3: eni1e3dd7d66d7@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether de:ab:83:56:db:2d brd ff:ff:ff:ff:ff:ff link-netns cni-5e405c0b-288d-6310-6693-0b9e4852be4d
    inet6 fe80::dcab:83ff:fe56:db2d/64 scope link 
       valid_lft forever preferred_lft forever
4: eni5f2d60c848c@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether 2a:72:31:82:22:ab brd ff:ff:ff:ff:ff:ff link-netns cni-66f3bf8e-6a35-30b3-00a0-933c28a5d345
    inet6 fe80::2872:31ff:fe82:22ab/64 scope link 
       valid_lft forever preferred_lft forever
5: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 02:40:04:29:de:eb brd ff:ff:ff:ff:ff:ff
    inet 172.31.10.238/20 brd 172.31.15.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::40:4ff:fe29:deeb/64 scope link 
       valid_lft forever preferred_lft forever
6: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 02:76:e9:96:26:c3 brd ff:ff:ff:ff:ff:ff
    inet 172.31.12.57/20 brd 172.31.15.255 scope global eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::76:e9ff:fe96:26c3/64 scope link 
       valid_lft forever preferred_lft forever
7: vlan1c357e15edb@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether 0e:1b:55:06:ef:90 brd ff:ff:ff:ff:ff:ff link-netns cni-7728e55d-b987-d6e8-f976-dc987e62903b
    inet6 fe80::c1b:55ff:fe06:ef90/64 scope link 
       valid_lft forever preferred_lft forever
8: vlan.eth.2@eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default qlen 1000
    link/ether 02:fc:a9:d7:13:2b brd ff:ff:ff:ff:ff:ff
    inet6 fe80::fc:a9ff:fed7:132b/64 scope link 
       valid_lft forever preferred_lft forever
9: vlan015fbd5badf@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether 16:e7:08:b0:13:63 brd ff:ff:ff:ff:ff:ff link-netns cni-4f3f82f3-5422-c275-bd38-ffb6139e82c2
    inet6 fe80::14e7:8ff:feb0:1363/64 scope link 
       valid_lft forever preferred_lft forever
10: vlan.eth.1@eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default qlen 1000
    link/ether 02:af:7e:67:89:e3 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::af:7eff:fe67:89e3/64 scope link 
       valid_lft forever preferred_lft forever
```

## ip netns

```bash
cni-4f3f82f3-5422-c275-bd38-ffb6139e82c2 (id: 3)
cni-7728e55d-b987-d6e8-f976-dc987e62903b (id: 2)
cni-66f3bf8e-6a35-30b3-00a0-933c28a5d345 (id: 1)
cni-5e405c0b-288d-6310-6693-0b9e4852be4d (id: 0)
```

## arp -n

```bash
Address                  HWtype  HWaddress           Flags Mask            Iface
172.31.12.149            ether   36:e1:dd:56:d4:53   C                     eni5f2d60c848c
172.31.0.1               ether   02:3f:d7:60:0f:ac   C                     eth0
169.254.169.254          ether   02:3f:d7:60:0f:ac   C                     eth0
172.31.3.129             ether   ae:a5:7d:45:9d:ac   C                     eni1e3dd7d66d7
172.31.2.224             ether   02:c3:38:f4:25:f7   C                     eth0

# after ping to pod IP, add more record
Address                  HWtype  HWaddress           Flags Mask            Iface
172.31.3.129             ether   ae:a5:7d:45:9d:ac   C                     eni1e3dd7d66d7
172.31.0.1               ether   02:3f:d7:60:0f:ac   C                     vlan.eth.2
172.31.9.169             ether   02:fc:a9:d7:13:2b   C                     eth0
172.31.0.1               ether   02:3f:d7:60:0f:ac   C                     vlan.eth.1
172.31.2.37              ether   02:af:7e:67:89:e3   C                     eth0
172.31.2.224             ether   02:c3:38:f4:25:f7   C                     eth0
169.254.169.254          ether   02:3f:d7:60:0f:ac   C                     eth0
172.31.12.149            ether   36:e1:dd:56:d4:53   C                     eni5f2d60c848c
172.31.9.169             ether   f2:ac:86:bb:65:bf   C                     vlan1c357e15edb 
172.31.2.37              ether   82:ca:9b:4b:45:45   C                     vlan015fbd5badf
172.31.0.1               ether   02:3f:d7:60:0f:ac   C                     eth0
```

## crictl ps

```bash
WARN[0000] runtime connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead. 
WARN[0000] image connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead. 
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD
c6e7a1a86c861       a606584aa9aa8       2 minutes ago       Running             nsenter             0                   0e52163c296e6       nsenter-n2qvfn
e0d75e518831c       099a2d701db1f       2 minutes ago       Running             nginx               0                   ccdc64c11eef5       my-deployment-758d6cbf7c-k6nvg
e2f2b5606378b       099a2d701db1f       2 minutes ago       Running             nginx               0                   0e695cadf0142       my-deployment-758d6cbf7c-n6b5v
ed02fdcedf3d8       2d6276296d8c3       8 minutes ago       Running             aws-eks-nodeagent   0                   76812b33054de       aws-node-mbmjp
7b3b0a9ec4180       86800e25303d1       8 minutes ago       Running             aws-node            0                   76812b33054de       aws-node-mbmjp
72b286443d223       5c4431bc8ea8c       12 minutes ago      Running             coredns             0                   f993beb562903       coredns-585d4c556b-z6ddv
2b953d5dc49fd       5c4431bc8ea8c       12 minutes ago      Running             coredns             0                   1c2b59eb49353       coredns-585d4c556b-z6qw6
8fc7beafe1ae7       df58ab4aa70ac       12 minutes ago      Running             kube-proxy          0                   fc0360444d705       kube-proxy-szr2x
```

## ip netns exec cni-4f3f82f3-5422-c275-bd38-ffb6139e82c2 ip a

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
3: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether 82:ca:9b:4b:45:45 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.31.2.37/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::80ca:9bff:fe4b:4545/64 scope link 
       valid_lft forever preferred_lft forever
```

## ip netns exec cni-4f3f82f3-5422-c275-bd38-ffb6139e82c2 arp -n

```bash
Address                  HWtype  HWaddress           Flags Mask            Iface
169.254.1.1              ether   16:e7:08:b0:13:63   CM                    eth0
```

## ip route

```bash
default via 172.31.0.1 dev eth0
169.254.169.254 dev eth0 
172.31.0.0/20 dev eth0 proto kernel scope link src 172.31.0.225 
172.31.3.129 dev eni1e3dd7d66d7 scope link 
172.31.12.149 dev eni5f2d60c848c scope link 
```

## ip rule

```bash
10:     from all iif vlan.eth.2 lookup 102
10:     from all iif vlan1c357e15edb lookup 102
10:     from all iif vlan.eth.1 lookup 101
10:     from all iif vlan015fbd5badf lookup 101
20:     from all lookup local
512:    from all to 172.31.3.129 lookup main
512:    from all to 172.31.12.149 lookup main
1024:   from all fwmark 0x80/0x80 lookup main
32766:  from all lookup main
32767:  from all lookup default
```

## ip r s t 101

```bash
default via 172.31.0.1 dev vlan.eth.1 
172.31.0.1 dev vlan.eth.1 scope link 
172.31.2.37 dev vlan015fbd5badf scope link 
```

## ip r s t 102

```bash
default via 172.31.0.1 dev vlan.eth.2 
172.31.0.1 dev vlan.eth.2 scope link 
172.31.9.169 dev vlan1c357e15edb scope link 
```

## iptables -S

```bash
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-N KUBE-EXTERNAL-SERVICES
-N KUBE-FIREWALL
-N KUBE-FORWARD
-N KUBE-KUBELET-CANARY
-N KUBE-NODEPORTS
-N KUBE-PROXY-CANARY
-N KUBE-PROXY-FIREWALL
-N KUBE-SERVICES
-A INPUT -m conntrack --ctstate NEW -m comment --comment "kubernetes load balancer firewall" -j KUBE-PROXY-FIREWALL
-A INPUT -m comment --comment "kubernetes health check service ports" -j KUBE-NODEPORTS
-A INPUT -m conntrack --ctstate NEW -m comment --comment "kubernetes externally-visible service portals" -j KUBE-EXTERNAL-SERVICES
-A INPUT -j KUBE-FIREWALL
-A FORWARD -m conntrack --ctstate NEW -m comment --comment "kubernetes load balancer firewall" -j KUBE-PROXY-FIREWALL
-A FORWARD -m comment --comment "kubernetes forwarding rules" -j KUBE-FORWARD
-A FORWARD -m conntrack --ctstate NEW -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A FORWARD -m conntrack --ctstate NEW -m comment --comment "kubernetes externally-visible service portals" -j KUBE-EXTERNAL-SERVICES
-A OUTPUT -m conntrack --ctstate NEW -m comment --comment "kubernetes load balancer firewall" -j KUBE-PROXY-FIREWALL
-A OUTPUT -m conntrack --ctstate NEW -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A OUTPUT -j KUBE-FIREWALL
-A KUBE-FIREWALL ! -s 127.0.0.0/8 -d 127.0.0.0/8 -m comment --comment "block incoming localnet connections" -m conntrack ! --ctstate RELATED,ESTABLISHED,DNAT -j DROP
-A KUBE-FORWARD -m conntrack --ctstate INVALID -j DROP
-A KUBE-FORWARD -m comment --comment "kubernetes forwarding rules" -m mark --mark 0x4000/0x4000 -j ACCEPT
-A KUBE-FORWARD -m comment --comment "kubernetes forwarding conntrack rule" -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

## iptables -S -t nat

```bash
-P PREROUTING ACCEPT
-P INPUT ACCEPT
-P OUTPUT ACCEPT
-P POSTROUTING ACCEPT
-N AWS-CONNMARK-CHAIN-0
-N AWS-SNAT-CHAIN-0
-N KUBE-KUBELET-CANARY
-N KUBE-MARK-MASQ
-N KUBE-NODEPORTS
-N KUBE-POSTROUTING
-N KUBE-PROXY-CANARY
-N KUBE-SEP-AGFPHYEWW47W3NWH
-N KUBE-SEP-ENNOY5MC2KBLEIWJ
-N KUBE-SEP-JUKZLL7LNAJLNFU6
-N KUBE-SEP-LWZLSIXIW4RDE2LP
-N KUBE-SEP-NCFKFHEKS34PD2FV
-N KUBE-SEP-O45UCR62K725X7JY
-N KUBE-SEP-RLATP7D5NCXSELBN
-N KUBE-SEP-UN7IU6VFECYWO4X5
-N KUBE-SEP-VWA7XYZO4DYZHGAT
-N KUBE-SEP-Y5ZAU3MTIUSLZPQE
-N KUBE-SERVICES
-N KUBE-SVC-ERIFXISQEP7F7OF4
-N KUBE-SVC-HLZB4VDDFP2JZATZ
-N KUBE-SVC-JD5MR3NA4I4DYORP
-N KUBE-SVC-NPX46M4PTMTKRN6Y
-N KUBE-SVC-TCOU7JCQXEZGVUNU
-A PREROUTING -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A PREROUTING -i eni+ -m comment --comment "AWS, outbound connections" -j AWS-CONNMARK-CHAIN-0
-A PREROUTING -m comment --comment "AWS, CONNMARK" -j CONNMARK --restore-mark --nfmask 0x80 --ctmask 0x80
-A OUTPUT -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A POSTROUTING -m comment --comment "AWS SNAT CHAIN" -j AWS-SNAT-CHAIN-0
-A AWS-CONNMARK-CHAIN-0 -d 172.31.0.0/16 -m comment --comment "AWS CONNMARK CHAIN, VPC CIDR" -j RETURN
-A AWS-CONNMARK-CHAIN-0 -m comment --comment "AWS, CONNMARK" -j CONNMARK --set-xmark 0x80/0x80
-A AWS-SNAT-CHAIN-0 -d 172.31.0.0/16 -m comment --comment "AWS SNAT CHAIN" -j RETURN
-A AWS-SNAT-CHAIN-0 ! -o vlan+ -m comment --comment "AWS, SNAT" -m addrtype ! --dst-type LOCAL -j SNAT --to-source 172.31.0.225 --random-fully
-A KUBE-MARK-MASQ -j MARK --set-xmark 0x4000/0x4000
-A KUBE-POSTROUTING -m mark ! --mark 0x4000/0x4000 -j RETURN
-A KUBE-POSTROUTING -j MARK --set-xmark 0x4000/0x0
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -j MASQUERADE --random-fully
-A KUBE-SEP-AGFPHYEWW47W3NWH -s 172.31.9.169/32 -m comment --comment "my-namespace/my-app" -j KUBE-MARK-MASQ
-A KUBE-SEP-AGFPHYEWW47W3NWH -p tcp -m comment --comment "my-namespace/my-app" -m tcp -j DNAT --to-destination 172.31.9.169:80
-A KUBE-SEP-ENNOY5MC2KBLEIWJ -s 172.31.12.149/32 -m comment --comment "kube-system/kube-dns:dns" -j KUBE-MARK-MASQ
-A KUBE-SEP-ENNOY5MC2KBLEIWJ -p udp -m comment --comment "kube-system/kube-dns:dns" -m udp -j DNAT --to-destination 172.31.12.149:53
-A KUBE-SEP-JUKZLL7LNAJLNFU6 -s 172.31.3.129/32 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-MARK-MASQ
-A KUBE-SEP-JUKZLL7LNAJLNFU6 -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp" -m tcp -j DNAT --to-destination 172.31.3.129:53
-A KUBE-SEP-LWZLSIXIW4RDE2LP -s 172.31.2.224/32 -m comment --comment "default/kubernetes:https" -j KUBE-MARK-MASQ
-A KUBE-SEP-LWZLSIXIW4RDE2LP -p tcp -m comment --comment "default/kubernetes:https" -m tcp -j DNAT --to-destination 172.31.2.224:443
-A KUBE-SEP-NCFKFHEKS34PD2FV -s 172.31.12.149/32 -m comment --comment "kube-system/kube-dns:metrics" -j KUBE-MARK-MASQ
-A KUBE-SEP-NCFKFHEKS34PD2FV -p tcp -m comment --comment "kube-system/kube-dns:metrics" -m tcp -j DNAT --to-destination 172.31.12.149:9153
-A KUBE-SEP-O45UCR62K725X7JY -s 172.31.3.129/32 -m comment --comment "kube-system/kube-dns:dns" -j KUBE-MARK-MASQ
-A KUBE-SEP-O45UCR62K725X7JY -p udp -m comment --comment "kube-system/kube-dns:dns" -m udp -j DNAT --to-destination 172.31.3.129:53
-A KUBE-SEP-RLATP7D5NCXSELBN -s 172.31.3.129/32 -m comment --comment "kube-system/kube-dns:metrics" -j KUBE-MARK-MASQ
-A KUBE-SEP-RLATP7D5NCXSELBN -p tcp -m comment --comment "kube-system/kube-dns:metrics" -m tcp -j DNAT --to-destination 172.31.3.129:9153
-A KUBE-SEP-UN7IU6VFECYWO4X5 -s 172.31.2.37/32 -m comment --comment "my-namespace/my-app" -j KUBE-MARK-MASQ
-A KUBE-SEP-UN7IU6VFECYWO4X5 -p tcp -m comment --comment "my-namespace/my-app" -m tcp -j DNAT --to-destination 172.31.2.37:80
-A KUBE-SEP-VWA7XYZO4DYZHGAT -s 172.31.12.149/32 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-MARK-MASQ
-A KUBE-SEP-VWA7XYZO4DYZHGAT -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp" -m tcp -j DNAT --to-destination 172.31.12.149:53
-A KUBE-SEP-Y5ZAU3MTIUSLZPQE -s 172.31.25.126/32 -m comment --comment "default/kubernetes:https" -j KUBE-MARK-MASQ
-A KUBE-SEP-Y5ZAU3MTIUSLZPQE -p tcp -m comment --comment "default/kubernetes:https" -m tcp -j DNAT --to-destination 172.31.25.126:443
-A KUBE-SERVICES -d 10.100.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp cluster IP" -m tcp --dport 53 -j KUBE-SVC-ERIFXISQEP7F7OF4
-A KUBE-SERVICES -d 10.100.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:metrics cluster IP" -m tcp --dport 9153 -j KUBE-SVC-JD5MR3NA4I4DYORP
-A KUBE-SERVICES -d 10.100.192.187/32 -p tcp -m comment --comment "my-namespace/my-app cluster IP" -m tcp --dport 80 -j KUBE-SVC-HLZB4VDDFP2JZATZ
-A KUBE-SERVICES -d 10.100.0.1/32 -p tcp -m comment --comment "default/kubernetes:https cluster IP" -m tcp --dport 443 -j KUBE-SVC-NPX46M4PTMTKRN6Y
-A KUBE-SERVICES -d 10.100.0.10/32 -p udp -m comment --comment "kube-system/kube-dns:dns cluster IP" -m udp --dport 53 -j KUBE-SVC-TCOU7JCQXEZGVUNU
-A KUBE-SERVICES -m comment --comment "kubernetes service nodeports; NOTE: this must be the last rule in this chain" -m addrtype --dst-type LOCAL -j KUBE-NODEPORTS
-A KUBE-SVC-ERIFXISQEP7F7OF4 -m comment --comment "kube-system/kube-dns:dns-tcp -> 172.31.12.149:53" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-VWA7XYZO4DYZHGAT
-A KUBE-SVC-ERIFXISQEP7F7OF4 -m comment --comment "kube-system/kube-dns:dns-tcp -> 172.31.3.129:53" -j KUBE-SEP-JUKZLL7LNAJLNFU6
-A KUBE-SVC-HLZB4VDDFP2JZATZ -m comment --comment "my-namespace/my-app -> 172.31.2.37:80" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-UN7IU6VFECYWO4X5
-A KUBE-SVC-HLZB4VDDFP2JZATZ -m comment --comment "my-namespace/my-app -> 172.31.9.169:80" -j KUBE-SEP-AGFPHYEWW47W3NWH
-A KUBE-SVC-JD5MR3NA4I4DYORP -m comment --comment "kube-system/kube-dns:metrics -> 172.31.12.149:9153" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-NCFKFHEKS34PD2FV
-A KUBE-SVC-JD5MR3NA4I4DYORP -m comment --comment "kube-system/kube-dns:metrics -> 172.31.3.129:9153" -j KUBE-SEP-RLATP7D5NCXSELBN
-A KUBE-SVC-NPX46M4PTMTKRN6Y -m comment --comment "default/kubernetes:https -> 172.31.2.224:443" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-LWZLSIXIW4RDE2LP
-A KUBE-SVC-NPX46M4PTMTKRN6Y -m comment --comment "default/kubernetes:https -> 172.31.25.126:443" -j KUBE-SEP-Y5ZAU3MTIUSLZPQE
-A KUBE-SVC-TCOU7JCQXEZGVUNU -m comment --comment "kube-system/kube-dns:dns -> 172.31.12.149:53" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-ENNOY5MC2KBLEIWJ
-A KUBE-SVC-TCOU7JCQXEZGVUNU -m comment --comment "kube-system/kube-dns:dns -> 172.31.3.129:53" -j KUBE-SEP-O45UCR62K725X7JY
```

## ls -al /proc/sys/net/ipv4/conf/

```bash
total 0
dr-xr-xr-x 1 root root 0 Jul 18 02:37 .
dr-xr-xr-x 1 root root 0 Jul 18 02:37 ..
dr-xr-xr-x 1 root root 0 Jul 18 02:37 all               # 1
dr-xr-xr-x 1 root root 0 Jul 18 02:37 default           # 1
dr-xr-xr-x 1 root root 0 Jul 18 03:03 eni1e3dd7d66d7    # 1
dr-xr-xr-x 1 root root 0 Jul 18 03:03 eni5f2d60c848c    # 1
dr-xr-xr-x 1 root root 0 Jul 18 03:03 eth0              # 2
dr-xr-xr-x 1 root root 0 Jul 18 03:03 eth1              # 1
dr-xr-xr-x 1 root root 0 Jul 18 03:03 eth2              # 1
dr-xr-xr-x 1 root root 0 Jul 18 03:03 lo                # 0
dr-xr-xr-x 1 root root 0 Jul 18 03:03 vlan015fbd5badf   # 1
dr-xr-xr-x 1 root root 0 Jul 18 03:03 vlan1c357e15edb   # 1
dr-xr-xr-x 1 root root 0 Jul 18 03:03 vlan.eth.1        # 1
dr-xr-xr-x 1 root root 0 Jul 18 03:03 vlan.eth.2        # 1
```

## ip route get 172.31.2.37

```bash
172.31.2.37 dev eth0 src 172.31.0.225 uid 0 
    cache
```

##

```bash

```

##

```bash

```

##

```bash

```

##

```bash

```

##

```bash

```
