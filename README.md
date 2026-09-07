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
