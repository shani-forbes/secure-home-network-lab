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

