# HomeLab-Simulations
This repository documents the architecture and operation of a custom, virtualized cybersecurity home lab. The environment was built to safely simulate real-world cyber attacks and practice network traffic analysis, threat hunting, and incident detection.


## 1. Environment Setup & Network Verification
The first step was to configure the virtual machines, assign IP addresses on the host-only network, and verify that the Intrusion Detection System (Security Onion) was properly monitoring the traffic.

This lab uses a fully isolated Internal Network to safely contain and monitor simulated attacks between the attacker and victim VMs. The Security Onion defense node is additionally configured with a Host-Only adapter, allowing secure management of the web console from the host PC without exposing the vulnerable environment to the local network.

### Attacker Setup
Configuring Kali Linux and verifying the attacker's assigned IP address on the network.

As shown in the output, the primary network interface (eth0) has not yet been assigned an IP address:
![K_ifconfig](https://github.com/user-attachments/assets/9d55ca7c-4335-4733-a343-bca1e1a8b1fa)

We use the *ifconfig* command to manually assign an IPv4 address to the eth0 interface, establishing connectivity for the virtual machine:
![K_inet_addr](https://github.com/user-attachments/assets/93010330-6935-4ed3-8a20-5cf58fc81614)

### Target Setup
Verifying the vulnerable Metasploitable 2 machine's network configuration so it could be targeted:
<img width="766" height="513" alt="ME_ifconfig" src="https://github.com/user-attachments/assets/f03185d5-0623-4afc-880c-0422dad30be0" />

Because the network interface is initially unconfigured, we use the *ifconfig* utility to manually assign a static IP address:
![ME_inet_addr](https://github.com/user-attachments/assets/4f01764a-ff59-4875-bf13-96f6972974f5)
