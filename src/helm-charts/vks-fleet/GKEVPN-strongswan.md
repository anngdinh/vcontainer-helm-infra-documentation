# GCP VPN - Strongswan

This guide will help you to configure Site-to-Site IPSec VPN on Ubuntu using Strongswan and Google Cloud VPN.

## Requirements

- Server with Ubuntu 22.04

## Example

- GCP VPN Gateway:
  - Public IP: _______GCP_PUBLIC_IP_______
  - Private Subnet: _______GCP_PRIVATE_SUBNET_______
- Strongswan Server:
  - Public IP: _______STRONGSWAN_PUBLIC_IP_______
  - Private IP: _______STRONGSWAN_PRIVATE_IP_______
  - Private Subnet: _______STRONGSWAN_PRIVATE_SUBNET_______

## Installations

```bash
sudo apt update && sudo apt upgrade -y
```

Add the following lines to the file:

```bash
sudo vim /etc/sysctl.conf

net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
net.ipv4.conf.all.accept_redirects = 0
```

Save and exit the file then run the following command to load settings

```bash
sudo sysctl -p
```

Install Strongswan

```bash
sudo apt install strongswan strongswan-pki libcharon-extra-plugins libcharon-extauth-plugins libstrongswan-extra-plugins libtss2-tcti-tabrmd0 -y
```

```bash
sudo systemctl enable strongswan-starter
```

```bash
systemctl status strongswan-starter
```

```bash
head -c 24 /dev/urandom | base64

# Output: ______STRONGSWAN_PSK______
```

Add the key to `/etc/ipsec.secrets` file

```bash
sudo vim /etc/ipsec.secrets

_______STRONGSWAN_PUBLIC_IP_______ _______GCP_PUBLIC_IP_______ : PSK "______STRONGSWAN_PSK______"
```

Backup ipsec configuration

```bash
sudo cp /etc/ipsec.conf /etc/ipsec.conf.bak
```

Create new configuration file

```bash
sudo vim /etc/ipsec.conf
```

Paste the following configuration

```bash
config setup
        charondebug="all"
conn site-b
        type=tunnel
        auto=start
        keyexchange=ikev2
        authby=secret
        right=_______GCP_PUBLIC_IP_______
        rightsubnet=_______GCP_PRIVATE_SUBNET_______
        left=_______STRONGSWAN_PRIVATE_IP_______
        leftid=_______STRONGSWAN_PUBLIC_IP_______
        leftsubnet=_______STRONGSWAN_PRIVATE_SUBNET_______
        ike=aes256-sha1-modp1024!
        esp=aes256-sha1!
        aggressive=no
        keyingtries=%forever
        ikelifetime=86400s
        lifetime=43200s
        lifebytes=576000000
        dpddelay=30s
        dpdtimeout=120s
        dpdaction=restart
```

Restart strongSwan

```bash
sudo ipsec restart
```

Check IPSec status

```bash
sudo ipsec statusall
```

Expected output:

```bash
Security Associations (1 up, 0 connecting):         # must be up
      site-a[2]: ESTABLISHED                        # must be established
```

Finally, add route inside your VPC network to route traffic to the Strongswan server if destination is GCP private subnet.

## Troubleshooting

- Check the status
- Use `tcpdump` to check the traffic
