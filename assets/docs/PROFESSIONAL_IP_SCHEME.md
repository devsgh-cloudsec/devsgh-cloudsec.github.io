# Professional Network IP Addressing Scheme
## Enterprise-Grade Design for Home Network

---

## IP Addressing Philosophy

**Design Principles:**
- Clean /24 boundaries for all networks
- Logical third octet indicates function
- Consistent fourth octet ranges for device types
- Room for growth in each segment
- Easy troubleshooting (IP tells you the network)
- VLAN IDs match third octet where possible

---

## PROFESSIONAL IP SCHEME

### Core Management Network
```
Network:  10.0.0.0/24
VLAN:     1 (Native/Untagged)
Gateway:  10.0.0.1 (pfSense)
Purpose:  Core infrastructure and management

IP Allocation:
  .1          - Gateway (pfSense LAN interface)
  .2-.9       - Network equipment
  .10-.49     - Servers & infrastructure
  .50-.99     - Management workstations
  .100-.199   - DHCP pool (if needed)
  .200-.254   - Reserved for expansion

Assigned Devices:
  10.0.0.1    - pfSense LAN Gateway
  10.0.0.2    - D9010 (Primary Admin Workstation)
  10.0.0.3    - pfSense VM (Nested testing)
  10.0.0.10   - Future: ESXi Management
  10.0.0.11   - B9010 (Secondary Workstation)
```

### Guest/Isolation Network
```
Network:  10.1.0.0/24
VLAN:     10 (Isolated)
Gateway:  10.1.0.1
Purpose:  Untrusted devices, guest access, neighbor assistance

IP Allocation:
  .1          - Gateway
  .100-.199   - DHCP pool (guests)
  .200-.254   - Reserved

Assigned Devices:
  10.1.0.1    - pfSense OPT1 Gateway
  10.1.0.100  - DHCP range start (guests/Elton)
  10.1.0.199  - DHCP range end
```

### WiFi/Family Network
```
Network:  10.10.0.0/24
VLAN:     10
Gateway:  10.10.0.1
Purpose:  Family WiFi devices, tablets, phones, laptops

IP Allocation:
  .1          - Gateway
  .2-.9       - WiFi infrastructure
  .10-.49     - Media devices
  .50-.99     - Static client assignments
  .100-.199   - DHCP pool (WiFi clients)
  .200-.254   - Reserved

Assigned Devices:
  10.10.0.1   - pfSense WiFi Gateway
  10.10.0.2   - D6400 WiFi Access Point
  10.10.0.10  - Media Center PC
  10.10.0.50  - Dev iPad (static)
  10.10.0.51  - Ishi iPad (static)
  10.10.0.52  - iPhone (static)
  10.10.0.53  - Galaxy Note5 (static)
  10.10.0.54  - Galaxy S7 (static)
  10.10.0.55  - Laptop (static)
  10.10.0.100 - DHCP range start
  10.10.0.199 - DHCP range end
```

### Storage/NAS Network
```
Network:  10.20.0.0/24
VLAN:     20
Gateway:  10.20.0.1
Purpose:  Network storage, backup, file servers

IP Allocation:
  .1          - Gateway
  .10-.49     - Storage devices
  .50-.99     - Backup infrastructure
  .100-.149   - Reserved for additional NAS
  .200-.254   - Reserved

Assigned Devices:
  10.20.0.1   - pfSense Storage Gateway
  10.20.0.10  - QNAS6 (Primary NAS)
  10.20.0.11  - QNAS8 (Secondary NAS)
  10.20.0.20  - ESXi-7 Server (Storage vMotion)
```

### IoT/Smart Home Network
```
Network:  10.30.0.0/24
VLAN:     30
Gateway:  10.30.0.1
Purpose:  IoT devices, smart home, isolated neighbor (Elton)

IP Allocation:
  .1          - Gateway
  .10-.49     - IoT devices
  .50-.99     - Smart home devices
  .100-.149   - DHCP pool (IoT)
  .150-.199   - Neighbor isolation zone (Elton)
  .200-.254   - Reserved

Assigned Devices:
  10.30.0.1   - pfSense IoT Gateway
  10.30.0.150 - Elton PC (when connected)
  10.30.0.100 - DHCP range start (IoT)
  10.30.0.149 - DHCP range end (IoT)
```

### Security/Camera Network
```
Network:  10.40.0.0/24
VLAN:     40
Gateway:  10.40.0.1
Purpose:  Security cameras, NVR, surveillance

IP Allocation:
  .1          - Gateway
  .10-.19     - NVR and recording infrastructure
  .20-.99     - IP cameras
  .100-.199   - DHCP pool (cameras)
  .200-.254   - Reserved

Assigned Devices:
  10.40.0.1   - pfSense Security Gateway
  10.40.0.10  - CCTV NVR (No Internet Access)
  10.40.0.20  - Camera 1
  10.40.0.21  - Camera 2
  10.40.0.22  - Camera 3
```

### Management VLAN
```
Network:  10.99.0.0/24
VLAN:     99
Gateway:  10.99.0.1
Purpose:  Network equipment management interfaces

IP Allocation:
  .1          - Gateway
  .2-.9       - Switches
  .10-.19     - Access points (management)
  .20-.29     - UPS/PDU
  .30-.49     - Other network equipment
  .50-.99     - Reserved
  .100-.199   - Reserved

Assigned Devices:
  10.99.0.1   - pfSense Management Gateway
  10.99.0.2   - GS108TV2-A Switch (Management)
  10.99.0.3   - GS108TV2-B Switch (Management)
  10.99.0.10  - D6400 AP (Management Interface)
  10.99.0.20  - UPS (if network-enabled)
```

### Switch Uplink Networks (Point-to-Point)
```
Network:  10.100.1.0/30
Purpose:  pfSense to GS108TV2-A Switch A uplink
  10.100.1.1  - pfSense OPT2 interface
  10.100.1.2  - GS108TV2-A switch IP

Network:  10.100.2.0/30
Purpose:  pfSense to GS108TV2-B Switch B uplink
  10.100.2.1  - pfSense OPT3 interface
  10.100.2.2  - GS108TV2-B switch IP
```

---

## ASCII Network Diagram

```
                                    INTERNET
                                        |
                                  [WAN: DHCP]
                                        |
                                  +----------+
                                  | pfSense  |
                                  | Firewall |
                                  +----------+
                                        |
        +-------------------------------+--------------------------------+
        |                               |                                |
   [10.0.0.1]                      [10.1.0.1]                      [10.100.1.1]
        |                               |                                |
   LAN: Core                    OPT1: Guest/Isolated            OPT2: Switch Uplink
   10.0.0.0/24                     10.1.0.0/24                    10.100.1.0/30
        |                               |                                |
   +----+----+                    +-----------+                   [10.100.1.2]
   |         |                    |           |                         |
D9010    pfSense VM          Elton PC    Guests                  GS108TV2-A
10.0.0.2  10.0.0.3          (10.30.0.150) (DHCP)                    Switch
                                                                        |
                                        +-------------------------------+
                                        |               |               |
                                   [10.10.0.1]    [10.20.0.1]     [10.99.0.2]
                                        |               |               |
                              VLAN 10: WiFi   VLAN 20: Storage  VLAN 99: Mgmt
                              10.10.0.0/24    10.20.0.0/24      10.99.0.0/24
                                        |               |
                            +-----------+       +-------+-------+
                            |                   |               |
                       D6400 AP             QNAS6           QNAS8
                       10.10.0.2          10.20.0.10     10.20.0.11
                            |
                +-----------+-----------+
                |           |           |
           Dev iPad    Ishi iPad   MediaCenter
          10.10.0.50  10.10.0.51   10.10.0.10


                                  [10.100.2.1]
                                        |
                                OPT3: Switch Uplink
                                   10.100.2.0/30
                                        |
                                  [10.100.2.2]
                                        |
                                  GS108TV2-B
                                    Switch
                                        |
                        +---------------+---------------+
                        |                               |
                   [10.30.0.1]                     [10.40.0.1]
                        |                               |
               VLAN 30: IoT/Elton            VLAN 40: Security
                10.30.0.0/24                    10.40.0.0/24
                        |                               |
              +---------+---------+             +-------+-------+
              |                   |             |               |
         IoT Devices         Elton Zone      NVR           Cameras
        (10.30.0.10+)      (10.30.0.150)  10.40.0.10   10.40.0.20+
```

---

## VLAN Summary Table

| VLAN ID | Network       | Gateway    | Description           | Trust Level |
|---------|---------------|------------|-----------------------|-------------|
| 1       | 10.0.0.0/24   | 10.0.0.1   | Core Management       | HIGH        |
| 10      | 10.10.0.0/24  | 10.10.0.1  | WiFi/Family           | MEDIUM      |
| 20      | 10.20.0.0/24  | 10.20.0.1  | Storage/NAS           | HIGH        |
| 30      | 10.30.0.0/24  | 10.30.0.1  | IoT/Neighbor Isolated | LOW         |
| 40      | 10.40.0.0/24  | 10.40.0.1  | Security/Cameras      | LOW         |
| 99      | 10.99.0.0/24  | 10.99.0.1  | Management            | HIGH        |
| N/A     | 10.1.0.0/24   | 10.1.0.1   | Guest/Test (No VLAN)  | ZERO        |

---

## Device Migration Table

### From Current Scheme → To Professional Scheme

| Device              | Current IP    | New IP       | Network          |
|---------------------|---------------|--------------|------------------|
| pfSense LAN GW      | 10.0.0.1      | 10.0.0.1     | CORE (unchanged) |
| D9010 (Admin PC)    | 10.0.0.2      | 10.0.0.2     | CORE (unchanged) |
| pfSense VM          | 10.0.0.3      | 10.0.0.3     | CORE (unchanged) |
| B9010               | 10.1.1.11     | 10.0.0.11    | CORE             |
| **WiFi Network**    |               |              |                  |
| D6400 AP            | 10.2.10.2     | 10.10.0.2    | WiFi VLAN 10     |
| Dev iPad            | 10.2.10.7     | 10.10.0.50   | WiFi VLAN 10     |
| Ishi iPad           | 10.2.10.6     | 10.10.0.51   | WiFi VLAN 10     |
| iPhone              | 10.2.10.4     | 10.10.0.52   | WiFi VLAN 10     |
| Galaxy Note5        | 10.2.10.3     | 10.10.0.53   | WiFi VLAN 10     |
| Galaxy S7           | 10.2.10.8     | 10.10.0.54   | WiFi VLAN 10     |
| Laptop              | 10.2.10.5     | 10.10.0.55   | WiFi VLAN 10     |
| Media Center        | 10.2.10.9     | 10.10.0.10   | WiFi VLAN 10     |
| **Storage**         |               |              |                  |
| QNAS6               | 10.2.20.2     | 10.20.0.10   | Storage VLAN 20  |
| QNAS8               | 10.2.20.12    | 10.20.0.11   | Storage VLAN 20  |
| ESXi-7              | 10.2.20.3     | 10.20.0.20   | Storage VLAN 20  |
| **IoT/Neighbor**    |               |              |                  |
| Elton PC            | 10.3.30.4     | 10.30.0.150  | IoT VLAN 30      |
| **Security**        |               |              |                  |
| NVR                 | 10.3.40.8     | 10.40.0.10   | Security VLAN 40 |
| **Management**      |               |              |                  |
| GS108TV2-A          | 10.2.2.2      | 10.99.0.2    | Management VLAN  |
| GS108TV2-B          | 10.3.3.2      | 10.99.0.3    | Management VLAN  |

---

## Why This Scheme is Better

### 1. **Clear Network Purpose**
- 10.0.x.x = Core/Management
- 10.10.x.x = WiFi/User devices (double 10 = user-facing)
- 10.20.x.x = Storage (20 = data)
- 10.30.x.x = IoT/Low trust (30 = isolated)
- 10.40.x.x = Security/Cameras (40 = surveillance)
- 10.99.x.x = Management VLAN (99 = network gear)
- 10.100.x.x = Point-to-point links

### 2. **Scalability**
- Each subnet can grow to 254 hosts
- Clean boundaries allow easy subnet expansion
- Unused ranges reserved for future needs

### 3. **Troubleshooting**
- IP address immediately tells you network function
- Consistent .1 for gateways
- Equipment always in .2-.9 range
- Servers in .10-.49 range
- Clients in .50-.99 range

### 4. **Security**
- Management VLAN separated
- IoT isolated with room for neighbor
- Point-to-point switch uplinks
- Clear trust boundaries

### 5. **Professional Standards**
- Follows RFC 1918 best practices
- Enterprise-style VLAN numbering
- Documentation-friendly
- Interview-worthy architecture

---

## Firewall Rule Implications

### New Inter-VLAN Rules Needed:

**WiFi (10.10.0.0/24) → Storage (10.20.0.0/24)**
- Allow: Plex, SMB, NFS from WiFi to NAS
- Block: Everything else

**Core (10.0.0.0/24) → All**
- Allow: Full access (administrative)

**IoT/Elton (10.30.0.0/24) → Internet Only**
- Allow: Internet (non-RFC1918)
- Block: All internal networks

**Security (10.40.0.0/24)**
- Allow: NVR to NAS (recording storage)
- Allow: Core to Security (view cameras)
- Block: Security to Internet (no phoning home)

**Management (10.99.0.0/24)**
- Allow: Core to Management (admin access)
- Block: All others to Management

---

## Implementation Notes

1. **Backup current config** before any changes
2. **Update pfSense interfaces** one at a time
3. **Reconfigure switch VLANs** with new IDs
4. **Update DHCP scopes** for new ranges
5. **Update static mappings** per migration table
6. **Test each network** before moving to next
7. **Update firewall rules** for new subnets
8. **Document changes** as you go

---

**This scheme is designed to impress in interviews and serve you for years to come!** 🎯
