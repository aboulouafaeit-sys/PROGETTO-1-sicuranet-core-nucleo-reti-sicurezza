# IP Addressing and VLAN Segmentation Plan

## 1. Network Subnet Allocations

| VLAN ID | Segment Name | Network Prefix | Subnet Mask | Default Gateway | Usable Host Range | Purpose |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- |
| **10** | `MANAGEMENT` | `192.168.10.0/24` | `255.255.255.0` | `192.168.10.1` | `192.168.10.2` - `192.168.10.254` | Switch & Router In-band Management (SSH) |
| **20** | `DATA_STAFF` | `192.168.20.0/24` | `255.255.255.0` | `192.168.20.1` | `192.168.20.11` - `192.168.20.254` | Administrative and staff workstations |
| **30** | `VOICE_VOIP` | `192.168.30.0/24` | `255.255.255.0` | `192.168.30.1` | `192.168.30.11` - `192.168.30.254` | Cisco 7960 IP Phones (VoIP Traffic) |
| **50** | `SERVERS_DC` | `192.168.50.0/24` | `255.255.255.0` | `192.168.50.1` | `192.168.50.2` - `192.168.50.254` | On-premises Servers (Static Addressing) |
| **60** | `FACTORY_IOT` | `192.168.60.0/24` | `255.255.255.0` | `192.168.60.1` | `192.168.60.11` - `192.168.60.254` | Smart TVs, Dashboards, and Factory Displays |

---

## 2. Dynamic Host Configuration Protocol (DHCP) Scopes

| Scope Pool Name | Subnet / Mask | Default Router | DNS Server | Special Options |
| :--- | :--- | :--- | :--- | :--- |
| `POOL-STAFF-DATA` | `192.168.20.0 /24` | `192.168.20.1` | `192.168.50.10` | None |
| `POOL-VOIP-VOICE` | `192.168.30.0 /24` | `192.168.30.1` | `192.168.50.10` | **Option 150:** `192.168.30.1` |
| `POOL-SMART-TVS` | `192.168.60.0 /24` | `192.168.60.1` | `8.8.8.8` | None |

> **Note on Option 150:** Cisco IP Phones require Option 150 during DHCP discovery to identify the TFTP server IP address where they can register and download their configuration/firmware XML profile.
