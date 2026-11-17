# Network Architecture & Design Overview

## 1. Architectural Model
This project follows the industry-standard Cisco Hierarchical Campus Design model tailored for a combined commercial office and industrial manufacturing facility:

1. **Edge / Core Layer:**
   - Centralized default gateways for all subnet segments.
   - Cisco CallManager Express (CME) running as an on-router PBX / IP telephony service.
   - DHCP server scopes providing IP addressing, default routes, DNS servers, and TFTP Option 150 indicators.
2. **Access Layer:**
   - High-density FastEthernet/GigabitEthernet switching with 802.1Q trunking.
   - Voice VLAN technology enabling converged voice and data over a single structured ethernet cable per desk.
   - Spanning Tree PortFast enabled on all host-facing ports for instantaneous DHCP address acquisition.

---

## 2. Voice and Data Convergence (VoIP Daisy-Chaining)

Connecting individual PCs and IP phones traditionally required two distinct cable runs to each employee desk. With Cisco IP Phone 7960 series devices, a 3-port integrated switch solves this:
- **Port 1 (Internal):** Communicates with the phone's internal DSP and VoIP firmware.
- **Port 2 (External - SW):** Connects to the wall outlet (Access Switch port).
- **Port 3 (External - PC):** Connects to the desktop computer.

### Traffic Segregation Logic:
- Voice packets are marked with 802.1p CoS priority and tagged with **VLAN 30** (802.1Q header).
- Computer data frames pass untagged into **VLAN 20**.
- The switch port configuration:
  ```cisco
  switchport mode access
  switchport access vlan 20
  switchport voice vlan 30
  ```

---

## 3. Industrial IoT & Smart Display Subsystem
Smart TVs and production displays in the factory zone are isolated into **VLAN 60**:
- Restricted from direct lateral access to administrative PCs.
- Direct connectivity to on-premise ERP/Database servers (**VLAN 50**) for real-time telemetry and KPI dashboard visualization.
- Internet access permitted for cloud firmware updates or remote diagnostics.
