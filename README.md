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

### Defender Setup.
Configuring Security Onion, specifically ensuring the sniffing interface (enp0s8) was properly attached to the network to capture traffic.
<img width="828" height="712" alt="SO_ip_a" src="https://github.com/user-attachments/assets/589dd1c0-0165-4f61-8b9d-5b18a70c47d5" />

Because the secondary network adapter (enp0s8) is configured as a slave to bond0, all IP addressing and network configurations must be applied directly to the bond0 master interface. Manual IP Configuration via *ifconfig*:
![SO_enp0s8_inet_addr](https://github.com/user-attachments/assets/74e8d588-2ea5-47a5-8483-1ac503eefc48)

### Connectivity Check.
Sending ICMP ping packets to verify that all machines could communicate and that the IDS could successfully observe the traffic before initiating any attacks.

Kali:
![K_ping_ME](https://github.com/user-attachments/assets/a1c73e11-4b39-49b1-a27b-18f069817914)

Security Onion:
![SO_ping](https://github.com/user-attachments/assets/46597764-7c9c-4577-9050-ee28b0652337)

## 2. Reconnaissance
Before exploiting the target, a network scan is required to identify open ports and vulnerable services. This tests the IDS's ability to detect scanning tools.

### Scanning the Target
Executing an aggressive Nmap scan from Kali against Metasploitable. This enumerated open ports and revealed the highly vulnerable vsftpd 2.3.4 FTP service:
![K_nmap](https://github.com/user-attachments/assets/f966a1a9-04c3-4aee-b8f7-f17720b2b47d)

### Detecting the Scan
Security Onion's Suricata engine successfully catching the scan, flagging the "Nmap Scripting Engine User-Agent" and generating high-severity alerts in the dashboard:
<img width="1838" height="921" alt="SO_nmap_detection" src="https://github.com/user-attachments/assets/4dcb1946-73a8-41b7-b2db-f9ebdbaa7f66" />

## 3. Stealth Exploitation (vsftpd Backdoor)
Using the information gathered from Nmap, a targeted exploit was launched against the vulnerable FTP service to gain a silent, persistent root shell.

### Weaponization
Launching the Metasploit Framework on Kali and configuring the unix/ftp/vsftpd_234_backdoor exploit module with the target's IP address:
![K_msfconsole](https://github.com/user-attachments/assets/e42c4ae9-b0a3-44f6-8612-bc2f6230148c)

### Exploitation
The exact moment the exploit executed the malicious FTP handshake (sending the :) trigger) and successfully opened a command shell on port 6200:
![K_opened_session_ME](https://github.com/user-attachments/assets/768be2ba-0262-44d5-90e6-31ebcdbe848a)

### Post-Exploitation
Executing commands (whoami, uname -a) in the root shell from Kali to confirm full system compromise:
![K_ME_hacked](https://github.com/user-attachments/assets/a8abc5ad-a7c4-400f-a977-5cc8fdc4e627)

### Threat Hunting
Navigating the Security Onion Hunt interface to filter for the backdoor traffic (destination port 6200) and using the Actions menu to pivot into the raw packet capture (PCAP) for deep inspection:
<img width="1840" height="915" alt="SO_pcap_action" src="https://github.com/user-attachments/assets/b94c7987-fd1b-4cfa-8f8d-d2022d52f511" />

### Forensic Proof
Analyzing the network transcript. This proves data exfiltration by showing the exact unencrypted commands (uname -a, whoami) and the server's responses (root) captured directly off the wire.
<img width="1840" height="915" alt="SO_pcap_analysis" src="https://github.com/user-attachments/assets/a8920421-dd04-43c1-b357-e250db87a36f" />

# 4. High-Volume Attack (FTP Brute Force)
To contrast the stealthy backdoor, a "noisy" brute-force attack was launched to test the IDS's ability to handle high-volume event generation and behavior-based detection.

### The Brute Force
Launching a dictionary attack using the THC-Hydra utility from Kali against the target's FTP service. This attempted over 1,000 passwords from the unix_passwords.txt wordlist in a matter of minutes:
![K_brute_force](https://github.com/user-attachments/assets/60d39899-1711-4164-86e3-ba9dbeded5be)

### The Alarm Bell
The Security Onion Alerts dashboard lighting up with a massive spike of "ET SCAN Potential FTP Brute-Force attempt" alerts, demonstrating behavior-based detection of repeated authentication failures:
<img width="1831" height="906" alt="SO_brute_force" src="https://github.com/user-attachments/assets/32539577-a516-4766-ac1b-86e15fc14818" />

### Investigating the Alert
Drilling down into a specific brute-force alert to analyze the metadata, including the attacker's source IP, destination IP, and the triggered Suricata rule:
<img width="1838" height="921" alt="SO_alert_details" src="https://github.com/user-attachments/assets/d5450985-dd49-436b-b95e-5195e28817e2" />

### The Raw Data
Viewing the underlying connection logs or PCAP data generated by the flood of failed FTP login attempts, wrapping up the forensic investigation of the attack:
<img width="1831" height="906" alt="SO_brute_force_PCAP" src="https://github.com/user-attachments/assets/8736f3f9-37c9-4895-85aa-a286e4f47cce" />

## 5. Conclusion
This home lab serves as a practical demonstration of both offensive security tactics and defensive network monitoring. It highlights the critical importance of SIEM solutions in detecting anomalous behavior and provides a foundational understanding of threat hunting and incident response.
