# AWS network

```bash
# k get pod -owide -A
NAMESPACE     NAME                                READY   STATUS    RESTARTS   AGE     IP              NODE                                               NOMINATED NODE   READINESS GATES
default       nsenter-5jrzw3                      1/1     Running   0          26m     172.31.23.106   ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
default       nsenter-gvq1fj                      1/1     Running   0          26m     172.31.23.106   ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
default       pause-pods-prefix-c4587fcb4-24fd2   1/1     Running   0          16s     172.31.9.248    ip-172-31-13-227.ap-southeast-2.compute.internal   <none>           <none>
default       pause-pods-prefix-c4587fcb4-6bpvt   1/1     Running   0          16s     172.31.2.37     ip-172-31-13-227.ap-southeast-2.compute.internal   <none>           <none>
default       pause-pods-prefix-c4587fcb4-864wk   1/1     Running   0          16s     172.31.16.74    ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
default       pause-pods-prefix-c4587fcb4-l9vvr   1/1     Running   0          16s     172.31.23.163   ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
default       pause-pods-prefix-c4587fcb4-nhnh4   1/1     Running   0          16s     172.31.20.235   ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
default       pause-pods-prefix-c4587fcb4-pcjns   1/1     Running   0          16s     172.31.10.237   ip-172-31-13-227.ap-southeast-2.compute.internal   <none>           <none>
default       pause-pods-prefix-c4587fcb4-qbtvx   1/1     Running   0          16s     172.31.21.208   ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
default       pause-pods-prefix-c4587fcb4-qqnzp   1/1     Running   0          16s     172.31.19.13    ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
default       pause-pods-prefix-c4587fcb4-sfsrf   1/1     Running   0          16s     172.31.10.85    ip-172-31-13-227.ap-southeast-2.compute.internal   <none>           <none>
default       pause-pods-prefix-c4587fcb4-sttvn   1/1     Running   0          16s     172.31.12.39    ip-172-31-13-227.ap-southeast-2.compute.internal   <none>           <none>
kube-system   aws-node-bx8hl                      2/2     Running   0          4m46s   172.31.13.227   ip-172-31-13-227.ap-southeast-2.compute.internal   <none>           <none>
kube-system   aws-node-d8pc9                      2/2     Running   0          6h31m   172.31.23.106   ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
kube-system   coredns-585d4c556b-47scf            1/1     Running   0          6h50m   172.31.20.89    ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
kube-system   coredns-585d4c556b-7sxzp            1/1     Running   0          6h50m   172.31.31.116   ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
kube-system   eks-pod-identity-agent-89d5d        1/1     Running   0          6h31m   172.31.23.106   ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
kube-system   eks-pod-identity-agent-prgbn        1/1     Running   0          4m46s   172.31.13.227   ip-172-31-13-227.ap-southeast-2.compute.internal   <none>           <none>
kube-system   kube-proxy-n9l4h                    1/1     Running   0          6h31m   172.31.23.106   ip-172-31-23-106.ap-southeast-2.compute.internal   <none>           <none>
kube-system   kube-proxy-t88cb                    1/1     Running   0          4m46s   172.31.13.227   ip-172-31-13-227.ap-southeast-2.compute.internal   <none>           <none>
```

## Node 1

- Public IPv4 address: 54.153.217.27
- Private IPv4 addresses
  - 172.31.23.93
  - 172.31.23.106
- Secondary private IPv4 addresses
  - 172.31.19.13
  - 172.31.16.74
  - 172.31.26.201
  - 172.31.27.54
  - 172.31.29.4
  - 172.31.20.235
  - 172.31.20.89
  - 172.31.31.116
  - 172.31.23.163
  - 172.31.21.208

```bash
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 0a:9b:38:f3:3d:6d brd ff:ff:ff:ff:ff:ff
    inet 172.31.23.106/20 brd 172.31.31.255 scope global dynamic eth0
       valid_lft 2141sec preferred_lft 2141sec
    inet6 fe80::89b:38ff:fef3:3d6d/64 scope link 
       valid_lft forever preferred_lft forever
3: dummy0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 8e:f2:23:9c:a6:7c brd ff:ff:ff:ff:ff:ff
4: pod-id-link0: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 7a:09:8c:86:4a:33 brd ff:ff:ff:ff:ff:ff
    inet 169.254.170.23/32 scope global pod-id-link0
       valid_lft forever preferred_lft forever
    inet6 fd00:ec2::23/128 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::7809:8cff:fe86:4a33/64 scope link 
       valid_lft forever preferred_lft forever
5: eni8ca2e41546c@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether 3a:26:c2:5b:d2:11 brd ff:ff:ff:ff:ff:ff link-netns cni-985a4f52-28eb-4b08-8146-3b8bc14f6369
    inet6 fe80::3826:c2ff:fe5b:d211/64 scope link 
       valid_lft forever preferred_lft forever
6: eni1b7730a1e08@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether 5a:d3:e7:07:5a:3f brd ff:ff:ff:ff:ff:ff link-netns cni-4663c64a-2a03-30da-7b8b-71e2c5ae267a
    inet6 fe80::58d3:e7ff:fe07:5a3f/64 scope link 
       valid_lft forever preferred_lft forever
7: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 0a:ba:0e:a0:e6:f3 brd ff:ff:ff:ff:ff:ff
    inet 172.31.23.93/20 brd 172.31.31.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::8ba:eff:fea0:e6f3/64 scope link 
       valid_lft forever preferred_lft forever
8: eni71f112c4c56@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether 46:6b:03:e7:2a:17 brd ff:ff:ff:ff:ff:ff link-netns cni-29b5491c-f30d-09ca-de23-59ab24813625
    inet6 fe80::446b:3ff:fee7:2a17/64 scope link 
       valid_lft forever preferred_lft forever
9: eni018c0830860@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether 1a:ba:ae:72:a7:be brd ff:ff:ff:ff:ff:ff link-netns cni-b2df6e62-aa23-bb52-1b65-83e0198e3396
    inet6 fe80::18ba:aeff:fe72:a7be/64 scope link 
       valid_lft forever preferred_lft forever
10: eni1964f948075@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether 42:94:e3:c1:c1:a5 brd ff:ff:ff:ff:ff:ff link-netns cni-4e0ae6c8-39e2-c857-336b-be4ffa69663c
    inet6 fe80::4094:e3ff:fec1:c1a5/64 scope link 
       valid_lft forever preferred_lft forever
11: enied57175cab5@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether c6:67:46:2a:8c:f0 brd ff:ff:ff:ff:ff:ff link-netns cni-f8f930b0-0e97-b354-bcc8-b461f63b2191
    inet6 fe80::c467:46ff:fe2a:8cf0/64 scope link 
       valid_lft forever preferred_lft forever
12: eni81b9597469c@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether 9a:fe:73:f9:e9:15 brd ff:ff:ff:ff:ff:ff link-netns cni-74093a17-65aa-d6ab-3461-6e28f1d1c4ee
    inet6 fe80::98fe:73ff:fef9:e915/64 scope link 
       valid_lft forever preferred_lft forever
13: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 0a:f0:85:1c:7e:8b brd ff:ff:ff:ff:ff:ff
    inet 172.31.18.110/20 brd 172.31.31.255 scope global eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::8f0:85ff:fe1c:7e8b/64 scope link 
       valid_lft forever preferred_lft forever

# ip route
default via 172.31.16.1 dev eth0 
169.254.169.254 dev eth0 
169.254.170.23 dev pod-id-link0 
172.31.16.0/20 dev eth0 proto kernel scope link src 172.31.23.106 
172.31.16.74 dev eni81b9597469c scope link 
172.31.19.13 dev eni1964f948075 scope link 
172.31.20.89 dev eni8ca2e41546c scope link 
172.31.20.235 dev eni71f112c4c56 scope link 
172.31.21.208 dev enied57175cab5 scope link 
172.31.23.163 dev eni018c0830860 scope link 
172.31.31.116 dev eni1b7730a1e08 scope link

# ip netns
cni-74093a17-65aa-d6ab-3461-6e28f1d1c4ee (id: 6)
cni-f8f930b0-0e97-b354-bcc8-b461f63b2191 (id: 5)
cni-4e0ae6c8-39e2-c857-336b-be4ffa69663c (id: 4)
cni-b2df6e62-aa23-bb52-1b65-83e0198e3396 (id: 3)
cni-29b5491c-f30d-09ca-de23-59ab24813625 (id: 2)
cni-4663c64a-2a03-30da-7b8b-71e2c5ae267a (id: 1)
cni-985a4f52-28eb-4b08-8146-3b8bc14f6369 (id: 0)

# iptables -S
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

# iptables -S -t nat
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
-N KUBE-SEP-4Q7QKBPQIGERYJJG
-N KUBE-SEP-5G5ZT6GHEYO6CPKS
-N KUBE-SEP-A7FK4LHINPNK7Q5J
-N KUBE-SEP-BSTQKTLB6FGFPDSM
-N KUBE-SEP-D5RN5V5QF2KONWHX
-N KUBE-SEP-GXZWWE44MTMMWHS2
-N KUBE-SEP-J43VA3HIT5MEHOVJ
-N KUBE-SEP-LJIAG6FIAF24EE5Y
-N KUBE-SERVICES
-N KUBE-SVC-ERIFXISQEP7F7OF4
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
-A AWS-SNAT-CHAIN-0 ! -o vlan+ -m comment --comment "AWS, SNAT" -m addrtype ! --dst-type LOCAL -j SNAT --to-source 172.31.23.106 --random-fully
-A KUBE-MARK-MASQ -j MARK --set-xmark 0x4000/0x4000
-A KUBE-POSTROUTING -m mark ! --mark 0x4000/0x4000 -j RETURN
-A KUBE-POSTROUTING -j MARK --set-xmark 0x4000/0x0
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -j MASQUERADE --random-fully
-A KUBE-SEP-4Q7QKBPQIGERYJJG -s 172.31.31.116/32 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-MARK-MASQ
-A KUBE-SEP-4Q7QKBPQIGERYJJG -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp" -m tcp -j DNAT --to-destination 172.31.31.116:53
-A KUBE-SEP-5G5ZT6GHEYO6CPKS -s 172.31.20.89/32 -m comment --comment "kube-system/kube-dns:dns" -j KUBE-MARK-MASQ
-A KUBE-SEP-5G5ZT6GHEYO6CPKS -p udp -m comment --comment "kube-system/kube-dns:dns" -m udp -j DNAT --to-destination 172.31.20.89:53
-A KUBE-SEP-A7FK4LHINPNK7Q5J -s 172.31.14.42/32 -m comment --comment "default/kubernetes:https" -j KUBE-MARK-MASQ
-A KUBE-SEP-A7FK4LHINPNK7Q5J -p tcp -m comment --comment "default/kubernetes:https" -m tcp -j DNAT --to-destination 172.31.14.42:443
-A KUBE-SEP-BSTQKTLB6FGFPDSM -s 172.31.20.89/32 -m comment --comment "kube-system/kube-dns:metrics" -j KUBE-MARK-MASQ
-A KUBE-SEP-BSTQKTLB6FGFPDSM -p tcp -m comment --comment "kube-system/kube-dns:metrics" -m tcp -j DNAT --to-destination 172.31.20.89:9153
-A KUBE-SEP-D5RN5V5QF2KONWHX -s 172.31.20.89/32 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-MARK-MASQ
-A KUBE-SEP-D5RN5V5QF2KONWHX -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp" -m tcp -j DNAT --to-destination 172.31.20.89:53
-A KUBE-SEP-GXZWWE44MTMMWHS2 -s 172.31.31.116/32 -m comment --comment "kube-system/kube-dns:metrics" -j KUBE-MARK-MASQ
-A KUBE-SEP-GXZWWE44MTMMWHS2 -p tcp -m comment --comment "kube-system/kube-dns:metrics" -m tcp -j DNAT --to-destination 172.31.31.116:9153
-A KUBE-SEP-J43VA3HIT5MEHOVJ -s 172.31.31.116/32 -m comment --comment "kube-system/kube-dns:dns" -j KUBE-MARK-MASQ
-A KUBE-SEP-J43VA3HIT5MEHOVJ -p udp -m comment --comment "kube-system/kube-dns:dns" -m udp -j DNAT --to-destination 172.31.31.116:53
-A KUBE-SEP-LJIAG6FIAF24EE5Y -s 172.31.37.201/32 -m comment --comment "default/kubernetes:https" -j KUBE-MARK-MASQ
-A KUBE-SEP-LJIAG6FIAF24EE5Y -p tcp -m comment --comment "default/kubernetes:https" -m tcp -j DNAT --to-destination 172.31.37.201:443
-A KUBE-SERVICES -d 10.100.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:metrics cluster IP" -m tcp --dport 9153 -j KUBE-SVC-JD5MR3NA4I4DYORP
-A KUBE-SERVICES -d 10.100.0.1/32 -p tcp -m comment --comment "default/kubernetes:https cluster IP" -m tcp --dport 443 -j KUBE-SVC-NPX46M4PTMTKRN6Y
-A KUBE-SERVICES -d 10.100.0.10/32 -p udp -m comment --comment "kube-system/kube-dns:dns cluster IP" -m udp --dport 53 -j KUBE-SVC-TCOU7JCQXEZGVUNU
-A KUBE-SERVICES -d 10.100.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp cluster IP" -m tcp --dport 53 -j KUBE-SVC-ERIFXISQEP7F7OF4
-A KUBE-SERVICES -m comment --comment "kubernetes service nodeports; NOTE: this must be the last rule in this chain" -m addrtype --dst-type LOCAL -j KUBE-NODEPORTS
-A KUBE-SVC-ERIFXISQEP7F7OF4 -m comment --comment "kube-system/kube-dns:dns-tcp -> 172.31.20.89:53" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-D5RN5V5QF2KONWHX
-A KUBE-SVC-ERIFXISQEP7F7OF4 -m comment --comment "kube-system/kube-dns:dns-tcp -> 172.31.31.116:53" -j KUBE-SEP-4Q7QKBPQIGERYJJG
-A KUBE-SVC-JD5MR3NA4I4DYORP -m comment --comment "kube-system/kube-dns:metrics -> 172.31.20.89:9153" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-BSTQKTLB6FGFPDSM
-A KUBE-SVC-JD5MR3NA4I4DYORP -m comment --comment "kube-system/kube-dns:metrics -> 172.31.31.116:9153" -j KUBE-SEP-GXZWWE44MTMMWHS2
-A KUBE-SVC-NPX46M4PTMTKRN6Y -m comment --comment "default/kubernetes:https -> 172.31.14.42:443" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-A7FK4LHINPNK7Q5J
-A KUBE-SVC-NPX46M4PTMTKRN6Y -m comment --comment "default/kubernetes:https -> 172.31.37.201:443" -j KUBE-SEP-LJIAG6FIAF24EE5Y
-A KUBE-SVC-TCOU7JCQXEZGVUNU -m comment --comment "kube-system/kube-dns:dns -> 172.31.20.89:53" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-5G5ZT6GHEYO6CPKS
-A KUBE-SVC-TCOU7JCQXEZGVUNU -m comment --comment "kube-system/kube-dns:dns -> 172.31.31.116:53" -j KUBE-SEP-J43VA3HIT5MEHOVJ

# arp -n
Address                  HWtype  HWaddress           Flags Mask            Iface
169.254.169.254          ether   0a:59:75:53:a7:1a   C                     eth0
172.31.21.208            ether   fa:18:ad:51:c7:1b   C                     enied57175cab5
172.31.20.89             ether   7e:66:05:68:72:3d   C                     eni8ca2e41546c
172.31.31.116            ether   be:cd:2a:2f:74:94   C                     eni1b7730a1e08
172.31.16.1              ether   0a:59:75:53:a7:1a   C                     eth0
```

```bash
# ip netns exec cni-b2df6e62-aa23-bb52-1b65-83e0198e3396 ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
3: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default 
    link/ether 8e:c5:8b:36:64:f4 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.31.23.163/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::8cc5:8bff:fe36:64f4/64 scope link 
       valid_lft forever preferred_lft forever

# ip netns exec cni-b2df6e62-aa23-bb52-1b65-83e0198e3396 ip route
default via 169.254.1.1 dev eth0 
169.254.1.1 dev eth0 scope link

# ip netns exec cni-b2df6e62-aa23-bb52-1b65-83e0198e3396 iptables -S
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
# ip netns exec cni-b2df6e62-aa23-bb52-1b65-83e0198e3396 iptables -S -t nat
-P PREROUTING ACCEPT
-P INPUT ACCEPT
-P OUTPUT ACCEPT
-P POSTROUTING ACCEPT

# ip netns exec cni-b2df6e62-aa23-bb52-1b65-83e0198e3396 ping 172.31.16.1
# ip netns exec cni-b2df6e62-aa23-bb52-1b65-83e0198e3396 arp -n
Address                  HWtype  HWaddress           Flags Mask            Iface
172.31.23.106            ether   1a:ba:ae:72:a7:be   C                     eth0
169.254.1.1              ether   1a:ba:ae:72:a7:be   CM                    eth0
```
