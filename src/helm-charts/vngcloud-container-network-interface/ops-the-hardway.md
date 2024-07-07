# OPS the hard way

## New port to network namespace

In OPS server

```bash
NETWORK_ID=90d74e7f-a8d0-4f52-adec-fd5bddd71f44
SERVER=annd2-2
# Create a new port
openstack port create --network $NETWORK_ID pod-port-2
openstack port set pod-port-2 --security-group IAAS_53539_1714970107604_Default

# Add port to server
openstack server add port $SERVER pod-port-2
```

In server

```bash
PORT_IP_1=172.16.1.7
PORT_IP_2=172.16.1.8
sudo ip netns add ns1
sudo ip link set eth1 netns ns1
sudo ip netns exec ns1 ip addr add $PORT_IP_2/24 dev eth1
sudo ip netns exec ns1 ip link set eth1 up

# Test
ip netns exec ns1 python3 -m http.server 8080
curl $PORT_IP_2:8080

# Test
python3 -m http.server 8080
ip netns exec ns1 curl $PORT_IP_1:8080
```
