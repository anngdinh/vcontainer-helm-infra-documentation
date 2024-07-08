# OPS the hard way

## New port direct to network namespace

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

# Clean up
ip netns del ns1
```

## New port -> veth pair -> network namespace

Inside the VM

```bash
PORT_IP_2=172.16.1.8
ip netns add ns1

# Create veth pair
ip link add veth0 type veth peer name veth1
ip link set veth1 netns ns1
ip netns exec ns1 ip addr add $PORT_IP_2/32 dev veth1


# set default arp
veth0_mac_add=$(cat /sys/class/net/veth0/address)
ip netns exec ns1 arp -i veth1 -s 169.254.1.1 $veth0_mac_add
# default is route with sample IP to interface veth1
ip netns exec ns1 ip link set veth1 up
ip netns exec ns1 ip route add 169.254.1.1 dev veth1 scope link
ip netns exec ns1 ip route add default via 169.254.1.1 dev veth1

# add route to pod
ip link set veth0 up
ip route add $PORT_IP_2/32 dev veth0 scope link

# Disable reverse path filtering
echo 0 | sudo tee /proc/sys/net/ipv4/conf/all/rp_filter
echo 0 | sudo tee /proc/sys/net/ipv4/conf/eth1/rp_filter

# # Enable IP forwarding
# sudo sysctl -w net.ipv4.ip_forward=1
# sudo ip netns exec ns1 sysctl -w net.ipv4.ip_forward=1

# Test
ip netns exec ns1 python3 -m http.server 8080

# Clean up
ip netns del ns1
```
