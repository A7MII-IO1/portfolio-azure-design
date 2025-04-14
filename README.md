# portfolio-azure-design

![คำอธิบายรูป](https://github.com/A7MII-IO1/portfolio-azure-design/blob/c9efe15f1673cf6ec93983a7cfdbed7860e993b7/diagram.png)

#### 1. Resource Group
<pre>
rg-azure
  - Region = (Asia Pacific) Southeast Asia
</pre>

#### 2. Virtual Network
<pre>
vnet-azure
  - Address Space = 192.168.0.0/16
  - Subnet
    - GatewaySubnet (192.168.0.0/24)
    - subnet-web(192.168.1.0/24)
    - subnet-app (192.168.2.0/24)
    - subnet-data (192.168.3.0)
    - subnet-mgmt (192.168.4.0/24)
  - DNS Servers = 10.0.1.1
</pre>
  
#### 3. Web Zone
<pre>
3.1 Network Security Group
      nsg-web
        - Inbound Security Rules
          - HTTP (80)
          - HTTPS (443)
          - RDP (3389)
            - Source = 192.168.4.0/24
        - Associate Subnet = subnet-web

3.2 Load Balancer
      lb-vmss-web
        - Type = public
        - Public IP address name = public-ip-web
        - Assignment = Static
        - Domain Name Label = autoscaleweb
        - Virtual Network = vnet-azure
        - Subnet = subnet-web
        - Backend pools
          - vmss-web (Instance 0) = VM-WEB1 (192.168.1.4)
          - vmss-web (Instance 1) = VM-WEB2 (192.168.1.5)
        - Health probes = tcp-probe (80)
        - Inbound NAT rules
            - rdp-web1
              - Destination = [Load balancer public ip]
              - Service = Custom (TCP/50001)
              - Target = vmss-web (Instance 0)(3389)
            - rdp-web2
              - Destination = [Load balancer public ip]
              - Service = Custom (TCP/50002)
              - Target = vmss-web (Instance 1)(3389)

3.3 DNS Zone
      chetniphat.co
      - Region = Southeast Asia
      - Record Set
      - chetniphat.co
        - Name = www
        - Type = A
        - IP Address = <public-ip-web>

3.4 Virtual Machine Scale Set
      vmss-web
        - OS disk image = WINS2019WEBIMAGE
        - Username = azureadmin
        - Password = Password1234
        - Instance count = 2
        - Scaling
          - Scale mode = Scale based on a metric
          - Rules
            - Scale out = Percentage CPU > 70 (average)
            - Scale In = Percentage CPU < 70 (average)
          - Instance limits
            - Minimun = 1
            - Maximum = 2
            - Default = 0

3.5 Virtual Machine
      VM-WEB1
        - Size = Standard DS1 v2
        - Image = image-wins2019-iis
        - OS Disk = Premium SSD
        - Virtual Network = vnet-azure
        - Subnet = subnet-web
      VM-WEB2
        - Size = Standard DS1 v2
        - Image = image-wins2019-iis
        - OS Disk = Premium SSD
        - Virtual Network = vnet-azure
        - Subnet = subnet-web
</pre>

#### 4. App Zone
<pre>
4.1 Network Security Group
      nsg-app
        - Inbound Security Rules
          - RDP (3389)
            - Source = 192.168.4.0/24
        - Associate Subnet = subnet-app

4.2 Load Balancer
      lb-vm-app
        - Region = Southeast Asia
        - Type = Internal
        - Virtual Network = vnet-azure
        - Subnet = subnet-app
        - Backend pools
          - VM-APP1 (192.168.2.4)
          - VM-APP2 (192.168.2.5)
          - VM-APP3 (192.168.2.6)
        - Health probes = tcp-probe [app port]
        - Inbound NAT rules
          - rdp-app1
            - Destination = [Load balancer public ip]
            - Service = Custom (TCP/50001)
            - Target = VM-APP1 (3389)
          - rdp-app2
            - Destination = [Load balancer public ip]
            - Service = Custom (TCP/50002)
            - Target = VM-APP2 (3389)
          - rdp-app3
            - Destination = [Load balancer public ip]
            - Service = Custom (TCP/50003)
            - Target = VM-APP3 (3389)

4.3 Availability Set
      as-app
      - Fault Domain = 2
      - Update Domain = 5
      - Virtual Machine
        - VM-APP1
        - VM-APP2
        - VM-APP3

4.4 Virtual Machine
      VM-APP1
        - Size = Standard DS1 v2
        - Availability Set = as-app
        - Image = Window Server 2019
        - OS Disk = Premium SSD
        - Virtual Network = vnet-azure
        - Subnet = subnet-app
      VM-APP2
        - Size = Standard DS1 v2
        - Availability Set = as-app
        - Image = Window Server 2019
        - OS Disk = Premium SSD
        - Virtual Network = vnet-azure
        - Subnet = subnet-app
      VM-APP3
        - Size = Standard DS1 v2
        - Availability Set = as-app
        - Image = Window Server 2019
        - OS Disk = Premium SSD
        - Virtual Network = vnet-azure
        - Subnet = subnet-app
</pre>

#### 5. Data Zone
<pre>
5.1 Network Security Group
      nsg-data
      - Inbound security rules
        - SQL (1433)
          - Source = 192.168.2.0/24
        - RDP (3389)
          - Source = 192.168.4.0/24
      - Associate Subnet = subnet-data

5.2 Load Balancer
      lb-vm-data
      - Region = Southeast Asia
      - Type = Internal
      - Virtual Network = vnet-azure
      - Subnet = subnet-data
      - Backend pools
        - VM-DB1 (192.168.3.4)
        - VM-DB2 (192.168.3.5)
        - VM-SHARE (192.168.3.6)
      - Health probes = tcp-probe (1433)
      - Inbound NAT rules
        - rdp-db1
          - Destination = [Load balancer public ip]
          - Service = Custom (TCP/50001)
          - Target = VM-DB1 (3389)
        - rdp-db2
          - Destination = [Load balancer public ip]
          - Service = Custom (TCP/50002)
          - Target = VM-DB2 (3389)
        - rdp-share
          - Destination = [Load balancer public ip]
          - Service = Custom (TCP/50003)
          - Target = VM-SHARE (3389)

5.3 Availability Set
      as-data
      - Fault Domain = 2
      - Update Domain = 5
      - Virtual Machine
        - VM-DB1
        - VM-DB2
        - VM-SHARE

5.4 Storage Account
sa-azure
- Performance = Premium
- Account Kind = StorageV2 (general purpose v2)
- Replication = LRS
- File Shares
  - Name = Report 2025
  - Quota = 1000GiB
  - Connect = VM-SHARE
  - Drive Letter = Z
  
5.5 Virtual Machine
      VM-DB1
        - Size = Standard DS1 v2
        - Availability Set = as-data
        - Image = Window Server 2019
        - OS Disk = Premium SSD
        - Virtual Network = vnet-azure
        - Subnet = subnet-data
      VM-DB2
        - Size = Standard DS1 v2
        - Availability Set = as-data
        - Image = Window Server 2019
        - OS Disk = Premium SSD
        - Virtual Network = vnet-azure
        - Subnet = subnet-data
      VM-SHARE
        - Size = Standard DS1 v2
        - Availability Set = as-data
        - Image = Window Server 2019
        - OS Disk = Premium SSD
        - Virtual Network = vnet-azure
        - Subnet = subnet-data
</pre>

#### 6. Management Zone
<pre>
6.1 Network Security Group
      nsg-mgmt
      - Inbound Security Rules
        - RDP (3389)
      - Associate Subnet = subnet-mgmt

6.2 Virtual Machine
      VM-DC2
        - Size = Standard DS1 v2
        - Image = Window Server 2019
        - OS Disk = Premium SSD
        - Virtual Network = vnet-azure
        - Subnet = subnet-mgmt
        - Domain = corp.internal
</pre>

#### 7. Site-to-Site VPN
<pre>
7.1 Virtual network gateway
      azure-nw-gw
      - Virtual network = azure-vnet
      - Subnet = GatewaySubnet (192.168.0.0/24)
      - Public IP address name = azure-vnet-gw-public-ip

7.2 Local network gateway
      local-nw-gw
      - IP address = [local server public ip]
      - Address space = 10.0.0.0/16

7.3 Connection
      vpn-azure-to-local
      - Connection Type = Site-to-site (IPSec)
      - Virtual network gateway = azure-nw-gw
      - Local network gateway = local-nw-gw
      - Preshared Key (PSK) = 1234
      - IKE Protocol = IKEv2
</pre>

#### 8. On-Premise
<pre>
8.1 Routing and Remote Access Service (RRAS)
      vpn-local-to-azure
      - Connection Type = VPN
      - VPN Type = KEv2
      - Destination Address = [Virtual network gateway public ip]
      - Protocol and Security = Route IP packets on this interface
      - Static Route
        - Destination = 192.168.0.0
        - Mask =  255.255.0.0
        - Metric = 10
      - Dial-Out Credentials
        - User name: azure
      - Preshared Key (PSK) = 1234

8.2 Virtual Machine
      VM-DC1
      - Domain = corp.internal
</pre>
