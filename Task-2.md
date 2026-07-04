# Network Reconnaissance and Vulnerability Assessment Report

## Title

**Network Reconnaissance, Port Scanning, Vulnerability Assessment, Packet Analysis, and Firewall Configuration in a Controlled Laboratory Environment**

---

# 1. Objective

The objective of this assessment was to gain practical experience with fundamental cybersecurity techniques in an authorized laboratory environment. The activities included passive reconnaissance, active reconnaissance, service discovery, vulnerability assessment, packet analysis, and firewall configuration using industry-standard security tools.

---

# 2. Scope

The assessment was conducted only against systems owned by the organization or intentionally deployed for security training.

**Testing Environment**

- Attacker Machine: Kali Linux
- Target Machine: Metasploitable2
- Virtualization Platform: VirtualBox/VMware
- Network Mode: Host-Only Network
- Tools Used:
    - WHOIS
    - NSLookup
    - Google Search Operators
    - Shodan
    - Nmap
    - Netcat
    - Telnet
    - OpenVAS (Greenbone)
    - Nessus Essentials
    - Wireshark
    - hping3
    - iptables

---

# 3. Passive Reconnaissance

## Objective

To collect publicly available information without directly interacting with the target system.

### 3.1 WHOIS Lookup

**Command**

```
whois google.com
```
![[Pasted image 20260704114134.png]]
**Purpose**

Obtain domain registration information including:

- Registrar
- Registration date
- Expiration date
- Name servers
- Administrative contacts (when publicly available)

**Observation**

WHOIS provides valuable ownership and DNS information useful during security assessments.

---

### 3.2 DNS Enumeration

**Command**

```
nslookup google.com
nslookup -type=A google.com
nslookup -type=MX google.com
nslookup -type=AAAA google.com
```
![[Pasted image 20260704114148.png]]
**Purpose**

Retrieve DNS records such as:

- A
- AAAA
- MX
- NS
- TXT

**Observation**

DNS information reveals mail servers, name servers, IPv4 addresses, IPv6 addresses, and other publicly available infrastructure details.

---

### 3.3 Google Search Operators

Examples

```
site:example.com

site:example.com filetype:pdf

site:example.com intitle:index.of
```
![[Pasted image 20260704114211.png]]
**Purpose**

Locate publicly indexed resources.

**Observation**

Search operators help identify publicly available documents and web pages that administrators may not realize are indexed.

---

### 3.4 Shodan Search

**Purpose**

Identify internet-facing services that have already been indexed.

**Observation**

Shodan provides information such as:

- Open ports
- Running services
- Software banners
- SSL certificates

---

# 4. Active Reconnaissance

## Objective

Discover hosts and identify running services within the authorized lab network.

---

### 4.1 Host Discovery

**Command**

```
nmap -sn 192.168.1.0/24
```

**Purpose**

Identify live hosts on the subnet.

**Observation**

Multiple active systems were successfully detected.

---

### 4.2 Banner Grabbing

**Command**

```
nc 192.168.56.101 22
```

or

```
telnet 192.168.56.101 25
```

**Purpose**

Identify services and software versions from server banners.

**Observation**

Banner information revealed service names and versions depending on server configuration.

---

# 5. Port and Service Scanning

## Objective

Identify open ports and determine running services.

---

### 5.1 TCP SYN Scan

**Command**

```
sudo nmap -sS 192.168.56.101
```

**Purpose**

Perform a TCP SYN scan to identify open TCP ports.

---

### 5.2 UDP Scan

**Command**

```
sudo nmap -sU 192.168.56.101
```

**Purpose**

Identify open UDP services.

---

### 5.3 Service and Operating System Detection

**Command**

```
sudo nmap -sV -O 192.168.56.101
```

**Purpose**

Detect:

- Running services
- Service versions
- Operating system

---

### 5.4 Report Export

**Command**

```
nmap -sS -sV -O 192.168.56.101 -oN nmap_report.txt
```

**Purpose**

Generate a text report for future analysis.
![[Screenshot From 2026-07-04 11-49-33.png]]
---

# 6. Vulnerability Assessment

Before scanning, both virtual machines must be placed on the same network adapter mode (such as **Host-Only** or **NAT Network** in VirtualBox/VMware) so they can communicate.

1. **Boot up the Metasploitable2 VM:**
    
    - Log in using the default credentials:
        
        - **Username:** `msfadmin`
            
        - **Password:** `msfadmin`
            
2. **Find the Target IP:** Run the following command inside the Metasploitable terminal:
    
    Bash
    
    ```
    ifconfig
    ```
    
    _Look for the `inet addr` field under `eth0` (e.g., `192.168.56.102` or `192.168.1.X`). Write this down._
    
3. **Test Connection from Kali Linux:** Open your terminal inside Kali Linux and try to ping the target IP to verify network reachability:
    
    Bash
    
    ```
    ping -c 3 192.168.56.101
    ```
    
    _If you see "64 bytes from..." packets returning successfully, your virtual network is configured correctly._

# Vulnerability Scanning

## 1. Environment Setup

Install one vulnerability scanner.

### Option A – OpenVAS (Greenbone Community Edition)

On Kali:

```
apt update
apt install gvm -y
```

Initialize:

```
gvm-setup
```

Start services:

```
gvm-start
```

Access:

```
https://127.0.0.1:9392
```

---

### Option B – Nessus Essentials

1. Download Nessus Essentials.
2. Install:

```
dpkg -i Nessus*.deb
```

Start:

```
 systemctl start nessusd
```

Open:

```
https://localhost:8834
```

Register with the free Essentials activation code.

---

## 2. Target VM

Use:

- Kali Linux
- Metasploitable2

Verify connectivity:

```
ping 192.168.56.101
```

Example:

```
192.168.56.101
```

---

## 3. Run the Scan

Create a new scan.

Target:

```
192.168.56.101
```

Typical checks include:

- Open ports
- Outdated services
- Weak configurations
- Known CVEs
- Default credentials
- Missing patches

---

## 4. Analyze Results

A typical Metasploitable2 report contains findings like:

|Severity|Examples|
|---|---|
|Critical|Remote code execution vulnerabilities|
|High|Outdated FTP, Samba, Apache|
|Medium|Weak SSL/TLS configuration|
|Low|Information disclosure, banners|

Record:

- CVE number
- Service
- Port
- Severity
- Suggested remediation

---

# Step 4: Packet Analysis Using Wireshark

## Capture Traffic

Open Wireshark.

Choose the interface connected to the lab network.

Click **Start Capture**.

Generate traffic:

- Browse an HTTP page
- Connect to an FTP server
- Perform a DNS lookup

---

## Useful Display Filters

HTTP

```
http
```

FTP

```
ftp
```

DNS

```
dns
```

TCP

```
tcp
```

UDP

```
udp
```

---

## Viewing Plaintext Credentials

Protocols such as FTP transmit credentials without encryption. In a lab, you can observe this to understand why encrypted alternatives are preferred.

Useful filters include:

```
ftp
```

or

```
ftp.request.command
```

You can also use **Follow TCP Stream** on an FTP session to inspect the protocol exchange.

Observe how usernames and passwords appear in plaintext in FTP traffic, illustrating why protocols like SFTP or FTPS are recommended.

---

## Packet Inspection

Look for:

- Source IP
- Destination IP
- TCP flags
- Sequence numbers
- Ports
- Payload

---

## Simulated DoS Observation

Within your own isolated lab, you can generate traffic for educational analysis and then observe it in Wireshark.

In the capture you should notice characteristics such as:

- Large numbers of TCP SYN packets
- Very few completed TCP handshakes
- Repeated attempts toward the same destination port
- High packet rates from one or more sources

These patterns help illustrate how abnormal traffic differs from normal client-server communication.

---

# Step 5: Firewall Basics

## View Existing Rules

```
sudo iptables -L -n -v
```

---

## Allow SSH

```
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

---

## Block Telnet

```
sudo iptables -A INPUT -p tcp --dport 23 -j DROP
```

---

## Allow HTTP

```
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

---

## Allow HTTPS

```
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

---
## Default Policy

```
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
```

---

## Save Rules

On Debian/Kali:

```
 apt install iptables-persistent
```

```
 netfilter-persistent save
```

---

## Sort and Analyze the Findings (Sub-Step C)

When the status indicator shows a checkmark (100% complete), click the scan name to enter its interactive monitoring board. You will be greeted with a color-coded chart mapping the vulnerabilities:

- 🔴 **Critical (CVSS 9.0–10.0):** Immediate exploit paths that allow complete system compromise. Look for critical entries like `vsftpd 2.3.4 Backdoor` or `UnrealIRCd Remote Code Execution`.
    
- 🟠 **High (CVSS 7.0–8.9):** High-severity structural flaws. Examples include legacy unpatched Apache web services, vulnerable SMB/Samba network mount points, or weak active SSH key pairs.
    
- 🟡 **Medium (CVSS 4.0–6.9):** Minor logic errors or configuration slip-ups, such as bad SSL/TLS protocol selections or directory traversal capability.
    
- 🔵 **Low / Info (CVSS 0.0–3.9):** Basic information leakage, active network service discovery banners, or ICMP replies.
---
# 7. Packet Analysis Using Wireshark

## Objective

 Packet Capture

Captured traffic while generating:

- HTTP
- FTP
- DNS

---
### 7.2 FTP Analysis

Display Filter

```
ftp
```

```
ftp.request.command=="USER"
```

---

### 7.3 TCP Stream Analysis

Using:

```
Follow → TCP Stream
```

### 7.4 SYN Flood Demonstration

A SYN flood simulation was performed in the isolated laboratory network using appropriate testing tools to observe the resulting increase in TCP SYN packets.

---

# 8. Firewall Configuration

Understand basic firewall rule creation using iptables.
### Allow SSH

```
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

---

### Block HTTP

```
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
```

---

### Port Scan Detection

Port-scan detection and response can be implemented using rate-limiting firewall rules or monitoring tools such as psad to identify aggressive scanning activity and respond according to organizational policy.

---

# 9. Findings

- Passive reconnaissance revealed publicly available domain and DNS information.
- Active reconnaissance successfully identified live hosts.
- Nmap detected multiple open ports and associated services.
- Vulnerability scanners identified security weaknesses with varying severity levels.
- Wireshark demonstrated differences between encrypted and unencrypted protocols.
- Firewall rules effectively controlled permitted and blocked network traffic.

---

# 11. Conclusion

The laboratory exercise demonstrated the complete lifecycle of a basic security assessment, including information gathering, host discovery, service identification, vulnerability assessment, network traffic analysis, and firewall configuration. Conducting these activities in a controlled environment reinforced foundational cybersecurity concepts while emphasizing the importance of authorization, responsible testing, and defensive best practices.