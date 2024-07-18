# OPS the hard way

## 1. New port direct to network namespace

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

Clean up

```bash
openstack port delete pod-port-2
```

## 2. New port -> veth pair -> network namespace

In OPS server

```bash
NETWORK_ID=90d74e7f-a8d0-4f52-adec-fd5bddd71f44
SERVER=annd2-main-2
# Create a new port
openstack port create --network $NETWORK_ID pod-port-22
openstack port set pod-port-22 --security-group IAAS_53539_1714970107604_Default

# Add port to server
openstack server add port $SERVER pod-port-22
```

Inside the VM

```bash
PORT_IP_2=172.16.1.16
ip link set eth1 up
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

# Enable IP forwarding, default is 0
sysctl net.ipv4.ip_forward
sudo sysctl -w net.ipv4.ip_forward=1
# sudo ip netns exec ns1 sysctl -w net.ipv4.ip_forward=1

# Test
ip netns exec ns1 python3 -m http.server 8080

# Clean up
ip netns del ns1
```

Clean up

```bash
openstack port delete pod-port-22
```

## 3. Trunk network

```bash
NETWORK_ID=90d74e7f-a8d0-4f52-adec-fd5bddd71f44
# Create a parent port
openstack port create --network $NETWORK_ID trunk-parent

# Create a trunk
openstack network trunk create --parent-port trunk-parent trunk1

# Create subports
openstack port create --network $NETWORK_ID subport1
openstack port create --network $NETWORK_ID subport2

# Add subports to trunk
openstack network trunk set \
  --subport port=subport1,segmentation-type=vlan,segmentation-id=100 \
  trunk1
openstack network trunk set \
  --subport port=subport2,segmentation-type=vlan,segmentation-id=200 \
  trunk1

# Show trunk
openstack network trunk show trunk1

# Set security group
openstack port set trunk-parent --security-group IAAS_53539_1714970107604_Default
openstack port set subport1 --security-group IAAS_53539_1714970107604_Default
openstack port set subport2 --security-group IAAS_53539_1714970107604_Default

# Add trunk to server
SERVER=annd2-super-main
SERVER_IP=172.16.1.4
SERVER_TEST=annd2-2
SERVER_TEST_IP=172.16.1.7
openstack server add port $SERVER trunk-parent
```

In Server
  
```bash
TRUNK_IP=172.16.1.5
ip addr add $TRUNK_IP/24 dev eth1
ip link set eth1 up

# Can't ping because server have 2 NICs, one for trunk, one for management
# Must add ip rule for trunk
echo "200 out" >> /etc/iproute2/rt_tables
ip route add default via $TRUNK_IP dev eth1 table out
ip rule add from $TRUNK_IP table out
ip route show table out
# can ping now

# # setup VLAN for subport https://www.baeldung.com/linux/vlans-create
# sudo apt-get install vlan
# sudo modprobe 8021q
# lsmod | grep 8021q

SUBPORT_1_MAC=02:69:dd:00:ee:19
SUBPORT_2_MAC=02:7b:1d:9f:80:5f
ip link add link eth1 address $SUBPORT_1_MAC name eth1.100 type vlan id 100
ip link set up eth1.100
ip link add link eth1 address $SUBPORT_2_MAC name eth1.200 type vlan id 200
ip link set up eth1.200
ip a
```

Create network namespace with subport

```bash
SUBPORT_1_IP=172.16.1.7
ip netns add ns1

# Create veth pair
ip link add veth0 type veth peer name veth1
ip link set veth1 netns ns1
ip netns exec ns1 ip addr add $SUBPORT_1_IP/32 dev veth1


# set default arp
veth0_mac_add=$(cat /sys/class/net/veth0/address)
ip netns exec ns1 arp -i veth1 -s 169.254.1.1 $veth0_mac_add
# default is route with sample IP to interface veth1
ip netns exec ns1 ip link set veth1 up
ip netns exec ns1 ip route add 169.254.1.1 dev veth1 scope link
ip netns exec ns1 ip route add default via 169.254.1.1 dev veth1
ip netns exec ns1 ip a
ip link set veth0 up
```

Config route (key part)

```bash
DEFAULT_GATEWAY=172.16.1.1
# table 101
ip route add $DEFAULT_GATEWAY dev eth1.100 scope link table 101
ip route add 172.16.1.7 dev veth0 scope link table 101
ip route add default via $DEFAULT_GATEWAY dev eth1.100 table 101
ip route show table 101


# rule
ip rule add from all iif eth1.100 table 101 prio 10
ip rule add from all iif veth0 table 101 prio 10


# test
ip netns exec ns1 python3 -m http.server 8080
tcpdump -nn icmp -l -i any

# clean up
ip netns del ns1
```

Clean up

```bash
openstack server remove port $SERVER trunk-parent

openstack network trunk unset --subport subport1 trunk1
openstack network trunk unset --subport subport2 trunk1
openstack port delete subport1
openstack port delete subport2
openstack network trunk delete trunk1
openstack port delete trunk-parent
```

## 4. Secgroup remote secgroup (continue from part 3)

- Can you config pod inside a node can access to other VMs but the node can't?
- When autoscale up, the pod should be able to access to the VMs.

```bash
# create a minimum security group, not allow any server can ping to it (in portal)

# create a empty security group for pod P
openstack security group create --description "for pod" POD_SEC

# create a security group DB allow from remote security group P
openstack security group create --description "for DB allow from pod" DB_SEC
openstack security group rule create \
    --remote-group POD_SEC \
    --ingress \
    DB_SEC

# attach security group DB to VMs
# attach security group P to pod
SERVER_DB_PORT=62df116d-c6ea-4500-ade9-e48b9e763e14
SERVER_POD_PORT=subport1
openstack port show $SERVER_DB_PORT
openstack port show $SERVER_POD_PORT

openstack port set $SERVER_DB_PORT --security-group DB_SEC
openstack port set $SERVER_POD_PORT --security-group POD_SEC

# test ping
SEVER_DB_IP=172.16.1.13
ping $SEVER_DB_IP
ip netns exec ns1 ping $SEVER_DB_IP
```
