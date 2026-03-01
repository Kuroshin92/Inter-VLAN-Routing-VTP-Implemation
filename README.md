# Inter-VLAN Routing & VTP Implementation

## 📌 Project Overview
This project demonstrates the implementation of **Router-on-a-Stick (ROAS)** architecture to enable communication between segmented departments. By using the **VLAN Trunking Protocol (VTP)** and **802.1Q encapsulation**, I established a scalable network that allows isolated VLANs to communicate over a single physical router interface.



## 🛠️ Technical Specifications
- **Routing:** Cisco IOS Subinterface configuration for Inter-VLAN routing.
- **Switching:** VTP Server/Client architecture for automated VLAN propagation.
- **Segmentation:**
  - VLAN 10 (R&D)
  - VLAN 20 (Engineering)
  - VLAN 30 (Sales)
  - VLAN 99 (Management/Native)

## 🚀 Key Configurations

### Router Subinterfaces (R1)
To handle multiple subnets on one port, I provisioned subinterfaces with specific 802.1Q tags:
- `Fa0/0.1`: 192.168.1.1 Router Gateway
- `Fa0/0.10`: 192.168.10.1 (VLAN 10)
- `Fa0/0.20`: 192.168.20.1 (VLAN 20)
- `Fa0/0.30`: 192.168.30.1 (VLAN 30)
- `Fa0/0.99`: 192.168.99.1 (Native VLAN).

### VTP Synchronization
Configured **SW2 as the VTP Server** to manage the global VLAN database and **SW1 as the VTP Client** to receive updates automatically, ensuring consistency across the fabric.

## 🔍 Troubleshooting & Resolution
During the verification phase, initial ICMP pings between PC1 and PC2 failed. I performed a root-cause analysis and identified two configuration gaps:

1. **Trunking Error:** The uplink from the switch to the router (Fa0/5) was not set to `switchport mode trunk`, preventing tagged traffic from reaching the gateway.
2. **Default Gateway Mismatch:** The switches were initially configured with an incorrect default gateway. Correcting the gateway to `192.168.99.1` (the subinterface IP for the Management VLAN) restored remote management connectivity.

## ✅ Results
- Successful end-to-end ICMP connectivity between all hosts across different VLANs.
- Automated VLAN propagation verified on all client switches using `show vlan brief`.
- Verified the Routing Table on R1 shows five active "C" (Connected) routes. `show ip routes`

## 💻 Configuration Highlights
# 1. Router-on-a-stick(Roas) Setup
To enable routing between VLANs using a single physical `interface (Fa0/0)`, I configured logical subinterfaces with 802.1Q encapsulation.
```
R1(config)# int fa0/0
R1(config-if)# no shutdown #Enable the physical path

# VLAN 10 (R&D) Subinterface
R1(config-if)# int fa0/0.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
#Same steps for VLAN 20 & 30 but added it to interface `fa0/0.20 and fa0/0.30` to keep it consistent.

#VLAN 99 (Management/Native) Subinterface
R1(config-if)# int fa0/0.99
R1(config-subif)# encapsulation dot1q 99 native
R1(config-subif)# ip address 192.168.99.1 255.255.255.0
```
# 2. VTP & Trunking Logic
I established a VTP domain named "Lab6" to automate VLAN management. I also manually configured the inter-switch links and the router uplink as trunks to carry multiple VLAN tags.
```
# SW2 (VTP Server) Configuration
SW2(config)# vtp mode server
SW2(config)# vtp domain lab6
SW2(config)# vtp password cisco

# Trunking Configuration (Applied to fa0/3 & fa0/4)
SW2(Config)# int range fa0/3-4
SW2(config-if-rang)# switchport mode trunk
SW2(config-if-rang)# switchport trunk native vlan 99 #switching the native vlan 1 to 99 for best security practice.
```
# 3. Management SVI & Default Gateway
To ensure the switches remained reachable for management from any subnet, I configured the Switch 
virtual interface (SVI) and the global default gateway.
```
# SW1 management Setup
SW1(config)# interface vlan 99
SW1(config-if)# ip address 192.168.99.11 255.255.255.0
SW1(config)# ip default-gateway 192.168.99.1 # Points to R1 subinterface
```
## 📊 Verification Commands 
These commands were used to validate the operational state of the network:
- `show vlan brief`: Confirmed VLANs 10,20,30, and 99 were distributed via VTP.
- `show ip route`: Verified that R1 populated its routing table with all five subnets.
- `show interface trunk`: Confirmed that Fa0/3, Fa0/4 (SW1 & SW2 port), and Fa0/5(SW1 port) were actively trunking.

## 🔧 Troubleshooting Log: Root Cause Analysis:
During the implementation phase, I encountered connectivity issues that prevented inter-VLAN communication. Below is the log of how these were identified and resolved.

| Problem | Symptoms | Root Cause  |  Resolution |
|---------|----------|-------------|-------------|
|**Inter-VLAN<br/>Ping Failure**|PC1 could not<br/>reach PC2 or PC3.|The uplink port(Fa0/5) from SW1<br/>to R1 was in access mode.|Reconfigured Fa0/5 as an **802.1Q trunk**<br/>to allow tagged traffic to<br/>reach the router.|
|**Management<br/>Reachability**|Could not ping<br/>SW1/SW2 from other<br/>subnets.|Default gateway on switches was set to `192.168.1.1`<br/>instead of the Management IP.|Updated `ip default-gateway`<br/>to `192.168.99.1` (the R1<br/>subinterface address for VLAN 99).|
