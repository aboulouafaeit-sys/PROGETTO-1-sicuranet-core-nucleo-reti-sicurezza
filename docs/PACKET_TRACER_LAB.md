# Enterprise & Factory Complete Packet Tracer Implementation Guide

This guide walks through assembling and interconnecting the complete enterprise topology in **Cisco Packet Tracer**, including the Next-Generation Firewall, IP Surveillance Cameras, Network Printers, Industrial Sensors, and VoIP CME telephony.

---

## 1. Hardware Inventory & Placement

| Device Type | Packet Tracer Model Name | Qty | Target Label | Role in Topology |
| :--- | :--- | :---: | :--- | :--- |
| **Firewall** | `ASA 5506-X` | 1 | `ASA-FIREWALL-01` | Perimeter Security, Zones (Outside, Inside, DMZ) |
| **Core Switch** | `3560-24PS` or `3650-24PS` | 1 | `CORE-DIST-SW01` | High-Speed Layer 3 Routing & SVIs |
| **Access Switch** | `2960-24TT` | 1 | `SW-ACCESS-OFFICE` | Office Desktops, VoIP, Printers, WLC |
| **Industrial Switch**| `2960-24TT` (or IE-2000) | 1 | `SW-ACCESS-FACTORY`| Factory Smart TVs, IP Cameras, Sensors, Door Access |
| **IP Phones** | `7960` | 3 | `Phone-101`, `102`, `103` | VoIP Extensions (Admin, Support, Factory) |
| **Workstations** | `PC-PT` | 2 | `PC-Admin`, `PC-Support` | Office end-user computers |
| **Printer** | `Printer-PT` | 1 | `Office-Printer-01` | Departmental Network Laser Printer |
| **Smart Display** | `Smart Device / TV` | 1 | `Factory-Smart-TV` | Overhead KPI Production Line Dashboard |
| **CCTV Cameras** | `IP Camera` (or PT-Camera) | 2 | `CAM-01-Office`, `CAM-02-Plant` | 24/7 Surveillance Video Capture |
| **IoT / OT Sensors**| `Thing` / `Door` / `Sensor` | 2 | `Door-Access-RFID`, `Temp-Sensor` | Industrial Facility Security & Monitoring |
| **Servers** | `Server-PT` | 3 | `SRV-DC-AD`, `SRV-NVR-CCTV`, `SRV-DMZ-WEB` | Active Directory, CCTV Storage, Public Web Server |
| **Wireless WLC** | `WLC-3504` or `WLC-2504` | 1 | `WLC-CORP` | Centralized Access Point Controller |
| **Access Points** | `LAP-PT` or `AccessPoint-PT`| 2 | `AP-Office`, `AP-Factory` | Dual-band Enterprise Wi-Fi coverage |

---

## 2. Comprehensive Cabling Guide

Use standard **Copper Straight-Through Cables** (solid black line) for all switch connections:

### A) Perimeter & Core Connections:
- `ASA-FIREWALL-01 Gig1/1` ➡️ `ISP Router / Cloud` (Outside Zone)
- `ASA-FIREWALL-01 Gig1/2` ➡️ `CORE-DIST-SW01 Gig0/1` (Inside Routed Link `10.0.0.1` ↔ `10.0.0.2`)
- `ASA-FIREWALL-01 Gig1/3` ➡️ `SRV-DMZ-WEB FastEthernet0` (DMZ Zone `172.16.90.10`)
- `CORE-DIST-SW01 Gig0/2` ➡️ `SW-ACCESS-OFFICE Gig0/1` (802.1Q Trunk)
- `CORE-DIST-SW01 Gig0/3` ➡️ `SW-ACCESS-FACTORY Gig0/1` (802.1Q Trunk)

### B) Office Switch Connections (`SW-ACCESS-OFFICE`):
- `Fa0/1` ➡️ `Phone-101 (Switch Port)` ➡️ `PC-Admin (PC Port)` *(Single Cable Daisy-Chain!)*
- `Fa0/2` ➡️ `Phone-102 (Switch Port)` ➡️ `PC-Support (PC Port)`
- `Fa0/11` ➡️ `Office-Printer-01 (FastEthernet0)` (VLAN 25)
- `Fa0/20` ➡️ `WLC-CORP (GigabitEthernet0/1)` (VLAN 40)
- `Fa0/21` ➡️ `AP-Office (Port 0)`

### C) Factory Switch Connections (`SW-ACCESS-FACTORY`):
- `Fa0/1` ➡️ `Phone-103 (Switch Port)` (Factory Extension 103)
- `Fa0/5` ➡️ `Factory-Smart-TV (FastEthernet0)` (VLAN 60)
- `Fa0/9` ➡️ `CAM-01-Office (FastEthernet0)` (VLAN 70)
- `Fa0/10` ➡️ `CAM-02-Plant (FastEthernet0)` (VLAN 70)
- `Fa0/17` ➡️ `Door-Access-RFID (FastEthernet0)` (VLAN 80)
- `Fa0/18` ➡️ `Temp-Sensor (FastEthernet0)` (VLAN 80)
- `Fa0/21` ➡️ `AP-Factory (Port 0)`

### D) Core Switch Server Farm Ports:
- `FastEthernet0/10` ➡️ `SRV-DC-AD (FastEthernet0)` (VLAN 50, IP `192.168.50.10`)
- `FastEthernet0/20` ➡️ `SRV-NVR-CCTV (FastEthernet0)` (VLAN 50, IP `192.168.50.50`)

---

## 3. Powering IP Phones & PoE Cameras in Packet Tracer

- **IP Phones (7960):** Click each phone -> select **Physical** tab -> drag the black power adapter from the lower right and drop it into the round DC power port.
- **IP Cameras:** If using standalone models that need an adapter, drag the power cord into the camera's power receptacle. When connected to PoE-enabled switch ports, they power on automatically.

---

## 4. Applying Configuration Files

1. **Firewall:** Open `ASA-FIREWALL-01` CLI ➡️ Paste [`configs/firewall_asa.cfg`](../configs/firewall_asa.cfg).
2. **Core Switch:** Open `CORE-DIST-SW01` CLI ➡️ Paste [`configs/switch_core_multilayer.ios`](../configs/switch_core_multilayer.ios).
3. **Office Switch:** Open `SW-ACCESS-OFFICE` CLI ➡️ Paste [`configs/switch_access_office.ios`](../configs/switch_access_office.ios).
4. **Factory Switch:** Open `SW-ACCESS-FACTORY` CLI ➡️ Paste [`configs/switch_access_factory.ios`](../configs/switch_access_factory.ios).

---

## 5. Verification Checklist

- [x] **VoIP Extension Dialing:** Phone 101 calls 103 on the factory floor; handset picks up with two-way audio.
- [x] **CCTV Video Stream:** Verify IP cameras acquire IP addresses in `192.168.70.0/24` and can reach the NVR server at `192.168.50.50`.
- [x] **Production TV Display:** Smart TV receives dynamic IP in `192.168.60.0/24` and pings internal web dashboard.
- [x] **Network Printing:** Workstation `192.168.20.11` sends test print packets to `192.168.25.11` across the Core Switch.
- [x] **Firewall Inspection:** Test internet access from PC 1; confirm traffic undergoes dynamic NAT (PAT) through ASA outside interface.
