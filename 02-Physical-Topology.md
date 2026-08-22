# PHYSICAL TOPOLOGY
## CMPG325-2026-104 | Bongani Nkosi | 37786458

---

## 1. PHYSICAL LAYOUT

### Building 1: Main Administration Building
- **Departments:** Admin, Finance, HR, IT
- **Devices:** 3x Access Switches (Cisco 2960)
- **Users:** ~41 staff

### Building 2: Public Services Centre
- **Department:** Public Services
- **Devices:** 1x Access Switch (Cisco 2960)
- **Users:** ~20 staff

### Building 3: IT/Server Room
- **Devices:** 1x Core Switch (3650), 1x Edge Router (4321)
- **Servers:** 2x Servers (App + File)

### Building 4: CCTV Building
- **Devices:** 1x Access Switch (2960), 8-12 Cameras
- **Purpose:** Security monitoring

---

## 2. DEVICE INVENTORY

| Device        | Model           | Quantity  | Location          | Purpose           |
|---------------|-----------------|-----------|-------------------|-------------------|
| Edge Router   | Cisco 4321      | 1         | Server Room       | NAT/Gateway       |
| Core Switch   | Cisco 3650      | 1         | Server Room       | L3 Routing        |
| Access Switch | Cisco 2960      | 4         | Various Buildings | End user access   |
| Server        | Generic Server  | 2         | Server Room       | App/File hosting  |
| PC            | Generic PC      | 75        | Various           | End users         |
| CCTV Camera   | IP Camera       | 8-12      | CCTV Building     | Security          |

---

## 3. CABLING PLAN

| Connection            | Medium        | Speed     |
|-----------------------|---------------|-----------|
| Core to Access        | Fiber         | 1 Gbps    |
| Access to End Devices | Copper (Cat6) | 100 Mbps  |
| Router to ISP         | Copper        | 100 Mbps  |