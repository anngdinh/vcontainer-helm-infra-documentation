# Container the hardway networking

## 1. rootns-netns

```bash
# create netns
ip netns add netns1

# create veth pair
ip link add veth0 type veth peer name veth1

# move veth1 to netns1
ip link set veth1 netns netns1

# set ip address for rootns = 201
ip addr add 10.0.0.201/24 dev veth0
ip link set veth0 up

# set ip address for netns1
ip netns exec netns1 ip addr add 10.0.0.101/24 dev veth1
ip netns exec netns1 ip link set veth1 up

# ping from rootns to netns1
ping 10.0.0.101

# ping from netns1 to rootns
ip netns exec netns1 ping 10.0.0.201

# cleanup
ip netns del netns1
```

## 2. rootns-netns-netns

```bash
# create netns
ip netns add netns1
ip netns add netns2

# create veth pair
ip link add veth0 type veth peer name veth1
ip link add veth2 type veth peer name veth3

# move veth1 to netns1
ip link set veth1 netns netns1
ip link set veth3 netns netns2

# set ip address for rootns = 201
ip addr add 10.0.0.201/24 dev veth0
ip addr add 10.0.1.201/24 dev veth2
ip link set veth0 up
ip link set veth2 up

# set ip address for netns1
ip netns exec netns1 ip addr add 10.0.0.101/24 dev veth1
ip netns exec netns1 ip link set veth1 up

# set ip address for netns2
ip netns exec netns2 ip addr add 10.0.1.101/24 dev veth3
ip netns exec netns2 ip link set veth3 up

# ping from rootns to netns1 and netns2
ping 10.0.0.101
ip netns exec netns1 ping 10.0.0.201
ping 10.0.1.101
ip netns exec netns2 ping 10.0.1.201

# ping from netns1 to netns2
ip netns exec netns1 ping 10.0.1.101
ip netns exec netns2 ping 10.0.0.101
# => ping: connect: Network is unreachable
ip netns exec netns1 ip route show
ip netns exec netns2 ip route show
# => default to reach outside of netns is missing
ip netns exec netns1 ip route add default via 10.0.0.201 dev veth1
ip netns exec netns2 ip route add default via 10.0.1.201 dev veth3
# But iptables is blocking FORWARD
iptables -P FORWARD ACCEPT
# ping should work now

# cleanup
ip netns del netns1
ip netns del netns2
```

## 3. rootns-netns-bridge-netns

```bash
# create netns
ip netns add netns1
ip netns add netns2

# create veth pair
ip link add veth0 type veth peer name veth1
ip link add veth2 type veth peer name veth3

# move veth1 to netns1
ip link set veth1 netns netns1
ip link set veth3 netns netns2

# create bridge
ip link add name br0 type bridge
ip addr add 10.0.0.1/24 dev br0
ip link set br0 up

# move veth0, veth2 to bridge
ip link set veth0 master br0
ip link set veth2 master br0
ip link set veth0 up
ip link set veth2 up

# set ip address for netns1
ip netns exec netns1 ip addr add 10.0.0.101/24 dev veth1
ip netns exec netns1 ip link set veth1 up
ip netns exec netns1 ip route add default via 10.0.0.1 dev veth1

# set ip address for netns2
ip netns exec netns2 ip addr add 10.0.0.102/24 dev veth3
ip netns exec netns2 ip link set veth3 up
ip netns exec netns2 ip route add default via 10.0.0.1 dev veth3

# ping rootns <=> netns1, netns2
ping 10.0.0.101
ping 10.0.0.102
ip netns exec netns1 ping 10.0.0.1
ip netns exec netns2 ping 10.0.0.1

# ping netns1 <=> netns2
ip netns exec netns1 ping 10.0.0.102
ip netns exec netns2 ping 10.0.0.101

# ping internet
ip netns exec netns1 ping 1.1.1.1 # not reachable
tcpdump -i br0 -nn icmp # => bridge is reacheable
tcpdump -i enp0s3 -nn icmp # => default is reacheable
# 10.0.0.101 > 1.1.1.1: ip of packet is can't resolve when return, should be source nat (change ip source)
gw_dev=$(ip -j route show | jq -r '.[]|select(.dst=="default")|.dev') # get default gateway device
echo $gw_dev
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o $gw_dev -j MASQUERADE # source nat, if packet from 10.0.0.0/24 and going out to $gw_dev, change ip source


# cleanup
ip netns del netns1
ip netns del netns2
ip link del br0
gw_dev=$(ip -j route show | jq -r '.[]|select(.dst=="default")|.dev') # get default gateway device
echo $gw_dev
iptables -t nat -D POSTROUTING -s 10.0.0.0/24 -o $gw_dev -j MASQUERADE
```

## 4. expose service to external

```bash
# create netns
ip netns add netns1

# create veth pair
ip link add veth0 type veth peer name veth1

# move veth1 to netns1
ip link set veth1 netns netns1

# set ip address for rootns = 201
ip addr add 10.0.0.201/24 dev veth0
ip link set veth0 up

# set ip address for netns1
ip netns exec netns1 ip addr add 10.0.0.101/24 dev veth1
ip netns exec netns1 ip link set veth1 up
ip netns exec netns1 ip route add default via 10.0.0.201 dev veth1

# expose service
ip netns exec netns1 python3 -m http.server 8080

# test from rootns
curl 10.0.0.101:8080 # should return index.html

# test from external
curl 10.0.0.101:8080 # not reachable
curl 192.168.56.12:80 # host ip, not reachable
# => need to forward port from host to netns
iptables -P FORWARD ACCEPT
iptables -t nat -A PREROUTING -d 192.168.56.12 -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.0.0.101:8080

# test from internal
curl 192.168.56.12:80 # host ip, not reachable
# => Failed to connect to 192.168.56.12 port 80 after 0 ms: Connection refused
# => need to forward port from host to netns
iptables -t nat -A OUTPUT -d 192.168.56.12 -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.0.0.101:8080

# cleanup
ip netns del netns1
iptables -t nat -D PREROUTING -d 192.168.56.12 -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.0.0.101:8080
iptables -t nat -D OUTPUT -d 192.168.56.12 -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.0.0.101:8080
```

## 5. load balancer

## 6. CNI

```bash
# Download CNI plugins
curl -OL https://github.com/containernetworking/plugins/releases/download/v1.5.0/cni-plugins-linux-amd64-v1.5.0.tgz
mkdir cni-bin
tar -C cni-bin -xzf cni-plugins-linux-amd64-v1.5.0.tgz
rm -rf cni-plugins-linux-amd64-v1.5.0.tgz

# start container without network
docker run -d --rm --network none --name alpine1 -t alpine:3.12

# check container network
docker exec alpine1 ip a # only loopback interface
docker exec alpine1 ip route show # nothing

# get container id and network namespace
container_id=$(docker inspect -f '{{.Id}}' alpine1) 
# 6bcebe665b31678906a5452fae78c1fd86a36a479c3054bb1c3e80aa05db826b
container_namespace=$(docker inspect alpine1 | jq -r '.[].NetworkSettings.SandboxKey') 
# /var/run/docker/netns/f0b18d774f4e

# config file
cat <<EOF > bridge.conf
{
    "cniVersion": "1.0.0",
    "name": "devops",
    "type": "bridge",
    "bridge": "devops0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "subnet": "10.0.0.0/24",
        "routes": [
            { "dst": "0.0.0.0/0" }
        ],
        "rangeStart": "10.0.0.2",
        "rangeEnd": "10.0.0.5",
        "gateway": "10.0.0.1"
    },
    "dns": {
        "nameservers": [ "1.1.1.1" ]
    }
}
EOF

# add network to container
CNI_PATH=$(pwd)/cni-bin \
    CNI_COMMAND=ADD \
    CNI_CONTAINERID=$container_id \
    CNI_NETNS=$container_namespace \
    CNI_IFNAME=eth10 \
    ./cni-bin/bridge <bridge.conf

# {
#     "cniVersion": "1.0.0",
#     "interfaces": [
#         {
#             "name": "devops0",
#             "mac": "5a:4e:aa:41:27:78"
#         },
#         {
#             "name": "veth568773e8",
#             "mac": "b2:7f:9c:ad:76:10"
#         },
#         {
#             "name": "eth10",
#             "mac": "4a:82:ea:14:6d:54",
#             "sandbox": "/var/run/docker/netns/f0b18d774f4e"
#         }
#     ],
#     "ips": [
#         {
#             "interface": 2,
#             "address": "10.0.0.2/24",
#             "gateway": "10.0.0.1"
#         }
#     ],
#     "routes": [
#         {
#             "dst": "0.0.0.0/0"
#         }
#     ],
#     "dns": {
#         "nameservers": [
#             "1.1.1.1"
#         ]
#     }
# }

########################## check container network
ip a
# 7: devops0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
#     link/ether 5a:4e:aa:41:27:78 brd ff:ff:ff:ff:ff:ff
#     inet 10.0.0.1/24 brd 10.0.0.255 scope global devops0
#        valid_lft forever preferred_lft forever
#     inet6 fe80::584e:aaff:fe41:2778/64 scope link 
#        valid_lft forever preferred_lft forever
# 8: veth568773e8@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master devops0 state UP group default 
#     link/ether e6:49:5f:a8:e8:85 brd ff:ff:ff:ff:ff:ff link-netnsid 0
#     inet6 fe80::b07f:9cff:fead:7610/64 scope link 
#        valid_lft forever preferred_lft forever

docker exec alpine1 ip a
# 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
#     link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
#     inet 127.0.0.1/8 scope host lo
#        valid_lft forever preferred_lft forever
#     inet6 ::1/128 scope host 
#        valid_lft forever preferred_lft forever
# 2: eth10@if8: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
#     link/ether 4a:82:ea:14:6d:54 brd ff:ff:ff:ff:ff:ff
#     inet 10.0.0.2/24 brd 10.0.0.255 scope global eth10
#        valid_lft forever preferred_lft forever
#     inet6 fe80::4882:eaff:fe14:6d54/64 scope link 
#        valid_lft forever preferred_lft 

docker exec alpine1 ip route show
# default via 10.0.0.1 dev eth10 
# 10.0.0.0/24 dev eth10 scope link  src 10.0.0.2

iptables -S -t nat
# -P PREROUTING ACCEPT
# -P INPUT ACCEPT
# -P OUTPUT ACCEPT
# -P POSTROUTING ACCEPT
# -N CNI-42e34ed26aeaeb1dcbad224f
# -N DOCKER
# -A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
# -A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
# -A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
# -A POSTROUTING -s 10.0.0.2/32 -m comment --comment "name: \"devops\" id: \"6bcebe665b31678906a5452fae78c1fd86a36a479c3054bb1c3e80aa05db826b\"" -j CNI-42e34ed26aeaeb1dcbad224f
# -A CNI-42e34ed26aeaeb1dcbad224f -d 10.0.0.0/24 -m comment --comment "name: \"devops\" id: \"6bcebe665b31678906a5452fae78c1fd86a36a479c3054bb1c3e80aa05db826b\"" -j ACCEPT
# -A CNI-42e34ed26aeaeb1dcbad224f ! -d 224.0.0.0/4 -m comment --comment "name: \"devops\" id: \"6bcebe665b31678906a5452fae78c1fd86a36a479c3054bb1c3e80aa05db826b\"" -j MASQUERADE
# -A DOCKER -i docker0 -j RETURN

docker exec alpine1 ping 1.1.1.1
# should be reachable

# cleanup
CNI_PATH=$(pwd)/cni-bin \
    CNI_COMMAND=DEL \
    CNI_CONTAINERID=$container_id \
    CNI_NETNS=$container_namespace \
    CNI_IFNAME=eth10 \
    ./cni-bin/bridge <bridge.conf
docker rm -f alpine1
```
