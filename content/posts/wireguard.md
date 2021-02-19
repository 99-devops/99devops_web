---
title: "What is wireguard on Azure, I'll do you better why is wireguard on Azure ?"
date: 2021-02-19T14:20:12+11:00
draft: false
description: "wireguard on azure cloud"
tags: [sre, networking]
categories: [sre]
---

![wg2](/img/wgbanner.png)


## What is WireGuard?

WireGuard is VPN protocol that uses state-of-the-art cryptography. It is fast yet simpler and better compared to IPsec and OpenVPN. Setting up WireGuard is supposed to be as simple as configuring SSH. Additionally, the use of excellent cryptographic technologies like Noise protocol framework, Curve25519, ChaCha20, Poly1305, BLAKE2, SipHash24, and HKDF makes is highly secure. Furthermore, its simplicity allows it to minimize its surface as it can be implemented with few codes. This in turn provides more secure environment as there is less change of errors.

## I'll do you better, why is wireguard ?

Although, various cloud providers have their own solution to for VPN we want to minimize the excessive reliance on the cloud. Also, it is easier to migrate the whole network to other platform or on-premises if necessary. Here, we will look after how-to setup point-to-site VPN from home to Azure Virtual Network.
Point-to-site VPN

## Where does WireGuard beat OpenVPN, IPSec and L2TP

| Wireguard   |      OpenVPN      |  IPsec IKEv2 |
|-------------|:-----------------:|-------------:|
| Very fast with little overhead and state-of-the-art cryptography |  Popular and open source but, not based on standards as it uses custom security protocols with SSL/TLS | Standard protocol for secure communication. Developed by Cisco and Microsoft | 
| Uses Curve25519, ChaCha20, Poly1305, and BLAKE2 protocols for encryption |   Uses OpenSSL library (AES, Camellia, Blowfish, 3DES) and TLS protocols for encryption | Uses algorithms such as AES, Camellia, Blowfish and 3DES for encryption
| No known major security vulnerabilities. Its smaller code base also enables easy audits for everyone. | No known major security vulnerabilities. But must be careful while implementing. Secure encryption algorithms should be implemented. |  No known major security vulnerabilities, however, leaked NSA presentation indicates it can be compromised. |
| Very high speed with low overhead | Can match IPSec if used with UDP connection instead of TCP | Faster than OpenVPN but does not match WireGuard |
| Can be configured on any port with UDP | Can be configured on any port with both TCP and UDP | 500 for initial key exchange, 4500 for NAT traversal, and 50 for IPSec encrypted data. |

## Setting up Site-to-site VPN in Azure Cloud using WireGuard

Now, we begin to create point-to-site VPN in Azure. We are using Azure CLI to create the necessary resources to set up the VPN. For this, go to Azure Portal and login to your account. Click on the upper right of the portal to open cloud shell.
![wg1](/img/wg1.png)

Then, select bash shell.
![wg2](/img/wg2.png)


1.	Create a resource group
```
az group create --name wireguardResourceGroup --location australiaeast
```

Check the resource group with command
```
az group list --query [].{Name:name} --output table
```


2.	Create a virtual network and check if vm has been created or not.
```
az network vnet create --name wireguardVpnNet --resource-group wireguardResourceGroup --subnet-name default
az network vnet list --query [].{Name:name} --output table
```


3.	Create a virtual machine and verify
```
az vm create \
    --resource-group wireguardResourceGroup \
    --name vpnServerVm \
    --image Openlogic:CentOs:8_2:latest\
    --vnet-name wireguardVpnNet \
    --subnet default \
    --admin-username azureuser \
    --generate-ssh-keys
```

```
az vm list --resource-group wireGuardResourceGroup --output table
```


4.	Create Network Security Group (NSG)
```
az network nsg create \
    --resource-group wireGuardResourceGroup \
    --location australiaeast \
    --name myNetworkSecurityGroup
```

5.	Create NSG Rule and Open SSH port
```
az network nsg rule create \
    --resource-group wireGuardResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name SSHRule \
    --protocol tcp \
    --priority 100 \
    --destination-port-range 22
```


6.	Open port for use by WireGuard
```
az network nsg rule create \
    --resource-group wireGuardResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name wireGuardNSGRule \
    --protocol udp \
    --priority 200 \
    --destination-port-range 51820
```


7.	Apply the NSG to the VM
```
az network nic update \
    --resource-group wireGuardResourceGroup \
    --name vpnServerVmVMNic \
    --network-security-group myNetworkSecurityGroup
```

8.	Get public ip of the vm
```
az vm show --resource-group wireGuardResourceGroup --name vpnServerVm --query {PublicIP:publicIps} -o table --show-details
```


9.	Connect to the VM using ssh
```
ssh azureuser@<public_ip_adress>
```


10.	Install and configure WireGuard and run following commands
```
sudo yum install elrepo-release epel-release
sudo yum install kmod-wireguard wireguard-tools
```

11. Reboot the computer and navigate to wireguard directory, generate public and private key, create configuration file, change the permission of prvatekey and wg0.conf file so that no other user has any permission and enable IP forwarding
```
sudo reboot
sudo su
cd /etc/wireguard
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
touch /etc/wireguard/wg0.conf
chmod 600 privatekey wg0.conf
nano /etc/sysctl.d/99-custom.conf
net.ipv4.ip_forward=1
sysctl -p /etc/sysctl.d/99-custom.conf
```

12. Open and add the following configuration to the wg0.conf file
```
vi wg0.conf

    [Interface]
    Address = 192.168.1.1/24
    SaveConfig = true
    Listenport = 51820
    PrivateKey = SERVER_PRIVATE_KEY
    PostUp     = firewall-cmd --zone=public --add-port 51820/udp && firewall-cmd --zone=public --add-masquerade
    PostDown   = firewall-cmd --zone=public --remove-port 51820/udp && firewall-cmd --zone=public --remove-masquerade

    [Peer]
    PublicKey = CLIENT_PUBLIC_KEY
    AllowedIps = 192.168.1.2/32
```

13.	Enable Firewalld service and enable wireguard on server.
```
service firewalld start
wg-quick up wg0
wg show wg0
systemctl enable wg-quick@wg0
```

14. Use the below configuration for the client
![wg3](/img/wg3.png)
```
    [Interface]
    PrivateKey = CLIENT_PRIVATE_KEY
    Address = 192.168.1.2/24
    DNS = 1.1.1.1

    [Peer]
    PublicKey = SERVER_PUBLIC_KEY
    AllowedIPs = 0.0.0.0/0
    Endpoint = [VPN_SERVER_PUBLIC_IP_ADDRESS]:51820
```
![wg4](/img/wg4.png)


15.	Cleaning up Azure resources
```
az group delete --name wireGuardResourceGroup
```