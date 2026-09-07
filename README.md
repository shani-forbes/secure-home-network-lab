# Building and Securing My Home Network

### Applying Security+ Fundamentals in a Real Environment

After earning my CompTIA Security+ certification, I wanted my next step to be more than moving on to another certification.

Security+ gave me a solid understanding of concepts like network segmentation, access control, secure configuration, firewalls, wireless security, and reducing attack surfaces. But understanding those concepts for an exam and actually implementing them are two different things.

I wanted to see what would happen when I had to apply those principles to a network I actually use.

This project documents the design, deployment, segmentation, security, troubleshooting, and validation of my home network as my first cybersecurity home lab.

## Project Overview

**Objective:** Build and secure a segmented home network while applying cybersecurity fundamentals in a real environment.

**Key Technologies:** TP-Link Omada · VLANs · ACLs · DHCP · Docker · Synology NAS · Wi-Fi 7 · TCP/IP

**Security Concepts:** Network Segmentation · Least Privilege · Access Control · Defense in Depth · Attack Surface Reduction · Secure Configuration

### Network Segmentation

| Network | VLAN | Subnet | Purpose |
|---|---:|---|---|
| Management | 1 | 192.168.0.0/24 | Network infrastructure and management |
| Trusted | 10 | 192.168.10.0/24 | Personal and trusted devices |
| IoT | 20 | 192.168.20.0/24 | Smart and IoT devices |
| Guest | 30 | 192.168.30.0/24 | Guest connectivity |

## Network Architecture

The network uses a segmented VLAN design with centralized routing and access control at the gateway. Both wireless access points broadcast the same SSIDs, allowing clients to connect through either the upstairs or downstairs AP while maintaining the same VLAN assignment and security policy.

![Home Network Architecture](assets/home-network-architecture.png.PNG)

## Environment & Technologies

The lab was built using a mix of physical networking hardware, self-hosted management software, and client devices across two physical locations.

### Core Infrastructure

- **TP-Link Omada ER707-M2** — gateway, firewall, VLAN routing, DHCP, NAT, and ACL enforcement
- **2 × TP-Link Omada EAP720** — Wi-Fi 7 access points
  - Upstairs AP connected directly to the gateway
  - Downstairs AP connected to the upstairs AP using wireless mesh backhaul
- **Synology NAS** — located at a separate physical site
- **Docker / Synology Container Manager** — used to host the Omada Software Controller
- **Omada Software Controller** — centralized management for the gateway and access points
- **Mac workstation** — used for configuration, troubleshooting, and network validation
- **iPhone** — used to test DHCP assignment, internet access, and inter-VLAN restrictions

## Self-Hosting the Omada Controller

Rather than purchasing a dedicated hardware controller, I chose to self-host the Omada Software Controller on an existing Synology NAS using Docker.

This added complexity to the project because the NAS was not located on the same local network as the Omada gateway and access points. As a result, the controller could not rely on normal Layer 2 discovery to find and adopt the devices.

The controller deployment also required troubleshooting several container-related issues, including directory mappings, persistent storage, startup errors, and rebuilding the container after correcting the data and log volume configuration.

Once the controller was running successfully, I used remote adoption and Omada cloud connectivity to bring the gateway and access points under centralized management.

This part of the project reinforced an important lesson: deploying a management platform is not just about getting the software to start. The controller has to be reachable, persistent, and correctly integrated with the network it manages.

## Designing the Network Segmentation

Once the core network was online, I wanted to reduce the amount of trust between different types of devices.

Instead of keeping every device on one flat network, I separated the environment into four logical networks based on function and risk.

### Management — VLAN 1

The Management network is reserved for infrastructure and administrative access.

**Subnet:** `192.168.0.0/24`

Typical devices and services include:

- Gateway and network infrastructure
- Wireless access points
- Management tools
- Administrative devices when needed

Separating management traffic helps reduce unnecessary exposure of infrastructure services to client devices.

### Trusted — VLAN 10

The Trusted network is used for personal and work devices that require normal access to the internet and selected internal resources.

**Subnet:** `192.168.10.0/24`

Typical devices include:

- Laptops
- Phones
- Personal computers
- Other trusted household devices

### IoT — VLAN 20

The IoT network is used for smart home and connected devices.

**Subnet:** `192.168.20.0/24`

Examples include:

- Smart appliances
- Cameras
- Smart plugs
- Televisions
- Other embedded or IoT devices

These devices generally require internet access but do not need access to trusted user devices or network management infrastructure.

### Guest — VLAN 30

The Guest network provides internet access for visitors without exposing internal resources.

**Subnet:** `192.168.30.0/24`

Guest clients are isolated from internal networks while still retaining internet connectivity.

## SSID-to-VLAN Mapping

The wireless networks were mapped directly to the appropriate VLANs:

| SSID | VLAN | Purpose |
|---|---:|---|
| FrancisOasis | 10 | Trusted devices |
| FrancisOasis-IOT | 20 | IoT devices |
| FrancisOasis-Guest | 30 | Guest devices |

Both access points broadcast the same SSIDs, so devices can associate with either the upstairs or downstairs AP while remaining in the same logical network.

This means physical location does not determine the security zone. The SSID and VLAN assignment do.

## Why Segmentation Alone Wasn't Enough

Creating separate VLANs was only the first step.

Because the gateway can route traffic between the VLANs, devices on different subnets could still communicate unless access controls were added.

That meant the next question was not simply:

**Which network is this device on?**

It became:

**What does this device actually need to access?**

That principle guided the firewall and ACL rules implemented next.

## Implementing Access Controls

Creating separate VLANs provided logical segmentation, but the gateway was still capable of routing traffic between those networks.

To turn that segmentation into an actual security boundary, I implemented gateway ACLs based on least privilege.

### IoT Access Controls

IoT devices need internet connectivity for normal operation, but there was no reason for them to initiate connections to personal devices or network infrastructure.

I created the following gateway ACL rules:

| Source | Destination | Action |
|---|---|---|
| IoT — VLAN 20 | Trusted — VLAN 10 | Deny |
| IoT — VLAN 20 | Management — VLAN 1 | Deny |

Internet access from the IoT network remained available.

This limits the potential impact of a compromised or insecure IoT device by preventing it from using the IoT network as a path to more sensitive parts of the environment.

### Guest Isolation

For the Guest network, I enabled Omada's built-in Guest Network isolation.

This allows guest devices to access the internet while preventing them from accessing private internal networks.

Rather than adding redundant ACLs that duplicated the same behavior, I used the platform's existing guest isolation control and validated that it produced the intended result.

### Applying Least Privilege

The goal was not to block traffic simply because I could.

For each network, I considered what its devices actually needed in order to function.

- **Trusted devices** require normal internet access and may need access to selected internal resources.
- **IoT devices** require internet access but do not need to initiate connections to Trusted or Management networks.
- **Guest devices** require internet access but should not have access to internal networks.
- **Management** is reserved for network infrastructure and administration.

This shifted the design from simply separating devices into different subnets to actively controlling communication between different trust zones.


## Testing & Validation

A configuration showing a `Deny` rule is not enough to prove that a security control is working.

After implementing the VLANs and ACLs, I tested the network from client devices to verify three things:

1. Devices were receiving addresses from the correct VLAN.
2. Restricted traffic could no longer cross security boundaries.
3. Internet connectivity continued to work after the restrictions were applied.

### Validating VLAN Assignment

I connected an iPhone to each wireless network and verified the DHCP-assigned IP address and default gateway.

| Network | Expected Subnet | Observed Address | Result |
|---|---|---|---|
| Trusted | `192.168.10.0/24` | `192.168.10.102` | Pass |
| IoT | `192.168.20.0/24` | `192.168.20.100` | Pass |
| Guest | `192.168.30.0/24` | `192.168.30.100` | Pass |

Each SSID correctly placed the client into its assigned VLAN while maintaining internet connectivity.

### Establishing a Baseline

Before applying the IoT ACLs, I tested whether a device connected to the IoT network could communicate with devices on other VLANs.

The IoT client was able to reach:

- A Mac connected to the Trusted network (`192.168.10.100`)
- A Mac interface on the Management network (`192.168.0.127`)

This confirmed an important point: creating separate VLANs had divided the network into different subnets, but inter-VLAN routing was still allowing communication between them.

### Testing IoT → Trusted

I then applied the ACL denying traffic from IoT (VLAN 20) to Trusted (VLAN 10).

I repeated the same connectivity test.

**Before ACL:** Reachable  
**After ACL:** 100% packet loss  
**Internet access:** Still available

**Result: PASS**

The IoT client could no longer initiate communication with a device on the Trusted network without losing the internet connectivity it required.

### Testing IoT → Management

I repeated the process for the Management network after applying the second ACL.

**Before ACL:** Reachable  
**After ACL:** 100% packet loss  
**Internet access:** Still available

**Result: PASS**

This confirmed that IoT devices could no longer initiate connections to the Management network.

### Testing Guest Isolation

I also tested a client connected to the Guest SSID.

The client received an address from the Guest subnet and retained internet connectivity, while attempts to reach private internal network addresses were unsuccessful.

**Result: PASS**

This validated the behavior of Omada's built-in Guest Network isolation.

### Validation Summary

| Test | Expected Result | Observed Result |
|---|---|---|
| Trusted client receives VLAN 10 address | `192.168.10.0/24` | Pass |
| IoT client receives VLAN 20 address | `192.168.20.0/24` | Pass |
| Guest client receives VLAN 30 address | `192.168.30.0/24` | Pass |
| IoT → Trusted before ACL | Reachable | Pass |
| IoT → Trusted after ACL | Blocked | Pass |
| IoT → Management before ACL | Reachable | Pass |
| IoT → Management after ACL | Blocked | Pass |
| IoT → Internet after ACL | Allowed | Pass |
| Guest → Internal networks | Blocked | Pass |
| Guest → Internet | Allowed | Pass |

The most useful part of this testing was comparing behavior before and after the security controls were applied.

Rather than assuming that segmentation or an ACL was working because it appeared correctly in the management interface, I established a baseline, implemented the control, repeated the test, and compared the results.

