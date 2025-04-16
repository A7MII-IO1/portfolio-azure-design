# Azure Architecture Design
![Azure](https://img.shields.io/badge/Azure-Cloud-blue?logo=microsoft-azure)
![OpenAI](https://img.shields.io/badge/OpenAI-ChatGPT_4-10a37f?logo=openai&logoColor=white)
![Notion](https://img.shields.io/badge/Notion-Workspace-000000?logo=notion&logoColor=white)
![draw.io](https://img.shields.io/badge/draw.io-diagram_tool-ff9900)

This repository contains a comprehensive Azure landing zone design based on the Hub-and-Spoke architecture model. It demonstrates network segmentation, high availability, and secure connectivity between cloud and on-premise environments, leveraging core Azure services such as Virtual Network, NSG, Load Balancer, VM Scale Set, and Site-to-Site VPN.

![Architecture Diagram](https://github.com/A7MII-IO1/portfolio-azure-design/blob/c9efe15f1673cf6ec93983a7cfdbed7860e993b7/diagram.png)

---

## Table of Contents

1. [Resource Group](#1️⃣-resource-group)
2. [Virtual Network](#2️⃣-virtual-network)
3. [Web Zone](#3️⃣-web-zone)
4. [App Zone](#4️⃣-app-zone)
5. [Data Zone](#5️⃣-data-zone)
6. [Management Zone](#6️⃣-management-zone)
7. [Site-to-Site VPN](#7️⃣-site-to-site-vpn)
8. [On-Premise](#8️⃣-on-premise)

---

## Resource Group
`Name: rg-azure`
```text
  - Region: Asia Pacific (Southeast Asia)
```

## Virtual Network
`Name: vnet-azure`
```text
  - Address Space: 192.168.0.0/16
  - Subnets:
    • GatewaySubnet (192.168.0.0/24)
    • subnet-web (192.168.1.0/24)
    • subnet-app (192.168.2.0/24)
    • subnet-data (192.168.3.0/24)
    • subnet-mgmt (192.168.4.0/24)
  - DNS Servers: 10.0.1.1
```

## Web Zone
### 3.1 Network Security Group
`Name: nsg-web`
```text
- Inbound Rules:
  • HTTP (80)
  • HTTPS (443)
- Associated Subnet: subnet-web
```

### 3.2 Load Balancer
`Name: lb-vmss-web`
```text
- Type: Public (Static IP)
- DNS Label: autoscaleweb
- Subnet: subnet-web
- Backend Pool:
  • VM-WEB1 (192.168.1.4)
  • VM-WEB2 (192.168.1.5)
- Health Probe: TCP on port 80
- NAT Rules:
  • rdp-web1 → TCP/50001 → VM-WEB1 (3389)
  • rdp-web2 → TCP/50002 → VM-WEB2 (3389)
```

### 3.3 DNS Zone
`Name: chetniphat.co`
```text
- Region: Southeast Asia
- Record Set: www.chetniphat.co → [public-ip-web]
```

### 3.4 Virtual Machine Scale Set
`Name: vmss-web`
```text
- Image: WINS2019WEBIMAGE
- Username: azureadmin
- Password: Password1234
- Instance Count: 2
- Scaling Rules:
  • Scale Out: CPU > 70%
  • Scale In: CPU < 70%
- Limits: min=1, max=2, default=0
```

### 3.5 Virtual Machines
`Name: VM-WEB1 & VM-WEB2`
```text
- Size: Standard DS1 v2
- Image: image-wins2019-iis
- OS Disk: Premium SSD
- Subnet: subnet-web
```

## App Zone
### 4.1 Network Security Group
`Name: nsg-app`
```text
- Subnet: subnet-app
```

### 4.2 Load Balancer
`Name: lb-vm-app`
```text
- Type: Internal
- Subnet: subnet-app
- Backend Pool:
  • VM-APP1, VM-APP2, VM-APP3
- Health Probe: TCP on app port
```

### 4.3 Availability Set
`Name: as-app`
```text
- Fault Domains: 2
- Update Domains: 5
- VMs: VM-APP1, VM-APP2, VM-APP3
```

### 4.4 Virtual Machines
`Name: VM-APP1, VM-APP2 & VM-APP3`
```text
- Image: Windows Server 2019
- Size: Standard DS1 v2
- OS Disk: Premium SSD
- Subnet: subnet-app
- Availability Set: as-app
```

## Data Zone
### 5.1 Network Security Group
`Name: nsg-data`
```text
- Allow SQL (1433) from 192.168.2.0/24
- Subnet: subnet-data
```

### 5.2 Load Balancer
`Name: lb-vm-data`
```text
- Type: Internal
- Subnet: subnet-data
- Backend Pool:
  • VM-DB1, VM-DB2, VM-SHARE
- Health Probe: TCP (1433)
```

### 5.3 Availability Set
`Name: as-data`
```text
- Fault Domains: 2
- Update Domains: 5
- VMs: VM-DB1, VM-DB2, VM-SHARE
```

### 5.4 Storage Account
`Name: sa-azure`
```text
- Performance: Premium
- Kind: StorageV2
- Replication: LRS
- File Share: Report 2025
  • Quota: 1000 GiB
  • Connected to VM-SHARE (Z: Drive)
```

### 5.5 Virtual Machines
`Name: VM-DB1, VM-DB2 & VM-SHARE`
```text
- Image: Windows Server 2019
- Size: Standard DS1 v2
- OS Disk: Premium SSD
- Subnet: subnet-data
- Availability Set: as-data
```

## Management Zone
### 6.1 Network Security Group
`Name: nsg-mgmt`
```text
- Subnet: subnet-mgmt
```

### 6.2 Virtual Machine
`Name: VM-DC2`
```text
- Image: Windows Server 2019
- Size: Standard DS1 v2
- Subnet: subnet-mgmt
- Domain: corp.internal
```

## Site-to-Site VPN
### 7.1 Virtual Network Gateway
`Name: azure-nw-gw`
```text
- Subnet: GatewaySubnet (192.168.0.0/24)
- Public IP: azure-vnet-gw-public-ip
```

### 7.2 Local Network Gateway
`Name: local-nw-gw`
```text
- IP Address: [local public IP]
- Address Space: 10.0.0.0/16
```

### 7.3 VPN Connection
`Name: vpn-azure-to-local`
```text
- Type: Site-to-Site (IPSec)
- Gateway: azure-nw-gw
- Local Gateway: local-nw-gw
- PSK: 1234
- IKE Protocol: IKEv2
```

## On-Premise
### 8.1 RRAS VPN
`Name: vpn-local-to-azure`
```text
- Type: VPN (IKEv2)
- Destination: [Azure VPN public IP]
- Static Route:
  • Destination: 192.168.0.0
  • Mask: 255.255.0.0
  • Metric: 10
- Dial-Out:
  • Username: azure
  • PSK: 1234
```

### 8.2 Virtual Machine
`Name: VM-DC1`
```text
- Domain: corp.internal
```

---

## Maintainer

**Chetniphat Varasai**  
Cloud Engineer  

---

## License

**This project is All Rights Reserved.** You are not permitted to use, distribute, or modify the code without explicit permission from the author.
