# Active Directory Identity Exploitation: Kerberoasting, Lateral Movement & SIEM Threat Hunting Lab

## 🌟 Introduction

In modern corporate environments, traditional firewalls and antivirus software are no longer enough to stop data breaches. Today, **Identity is the new security perimeter**. Real-world hackers rarely use cinematic "exploits" to smash through a digital wall; instead, they steal legitimate user passwords and simply log into the network completely undetected. This project simulates this exact tactical landscape.

This project marks a comprehensive, hands-on expansion of my enterprise telemetry environment into the realm of **Identity Threat Detection and Response (ITDR)**. The purpose of this lab was to design a realistic corporate network infrastructure, deliberately introduce common administrative oversights, and execute a multi-stage cyberattack lifecycle. 

By pivoting seamlessly between an offensive threat actor and an enterprise security defender, I engineered a project that maps the physics of an identity compromise directly to advanced **Splunk SIEM** behavioral analytics. This lab serves as hard engineering proof of my ability to harden corporate directory assets, analyze complex authentication flows, and hunt down stealthy, credential-based intrusions completely in the dark.

---

## 📌 Project Overview
Welcome to my enterprise security simulation lab! In this project, I built a corporate network sandbox from scratch to simulate a multi-stage cyberattack. I played both roles: the **Red Team (the attacker)** trying to sneak through the network, and the **Blue Team (the defender)** using a central security dashboard to catch the hacker red-handed.

The goal of this project was to understand **Identity Theft** inside a corporate network and engineer advanced security monitoring to make hidden hackers visible.

---

## 🏗️ The Network Sandbox Architecture
To make this simulation mirror a real corporation, I engineered a 4-node network laboratory using isolated virtual machines:
1. **Windows Server (Domain Controller - `ADDC01`):** The central "brain" of the company. It manages all user accounts, passwords, and security rules.
2. **Windows 10 Workstation (`HR-DESKTOP`):** A standard employee desktop computer used by our victim persona, a developer named Mike Daniels.
3. **Kali Linux (Attacker Machine):** A rogue machine sitting on the network wires used to launch scanning and exploitation tools.
4. **Linux Server (Splunk SIEM Dashboard):** The security tower. It acts as our digital video recorder, collecting security logs from every computer on the network so we can search for bad behavior.

---

## 🗺️ The Chronological Story: How the Attack Works

### 🧱 Phase 1: Hardening the Security Cameras (Defensive Baseline)
By default, Windows leaves its most advanced security tracking turned **off** to save computer memory. If an attacker sneaks in using stolen credentials, they remain completely invisible.
* **What I did:** I logged into the Domain Controller and modified the **Group Policy** to turn on Advanced Auditing. I forced the system to record every single time a network login occurred or a digital credential ticket was requested. 
* **The Result:** This laid the trap. The exact millisecond a hacker tried to touch a password file, Windows was forced to write it down.

### 🧱 Phase 2: Mapping the Building (Attacker Reconnaissance)
Imagine a burglar wanting to rob an office building. They don't just kick the door open blindly; they map out the facility first. 
* **What I did:** From the Kali Linux attack machine, I used a tool called **NetExec** to sweep the network. 
* **The Discovery:** The tool handed me a complete blueprint of the network, showing me exactly where the Domain Controller was sitting, the names of the employee desktops (`HR-DESKTOP`), and exposing that the workstations had their digital security seals turned off (`signing:false`).

### 🧱 Phase 3: The Kerberoasting Exploit (Privilege Escalation)
In corporate networks, computers don't ask you for your password every time you click a file; instead, they give you automated digital vouchers (called Kerberos Tickets). 
* **The Flaw:** Active Directory will hand out a service ticket to *any* employee who asks for it. 
* **What I did:** I simulated a scenario where a standard employee (Mike Daniels) had his account compromised. Using his low-privilege access, I ran an attack script called **Impacket** to ask the Domain Controller for the ticket belonging to the highly sensitive **SQL Database Service Account (`svc-sql`)**. 
* **The Theft:** The Domain Controller complied and sent the encrypted ticket back to my Kali machine. I copied that encrypted text block, put it into an offline password-cracking tool called **John the Ripper**, and successfully guessed the database password in less than a second, unmasking the plain-text password: `DbSecurePass2026!`.

### 🧱 Phase 4: Lateral Movement (Crossing the Network Bridge)
Now that the attacker has stolen the database service password, they want to see if the network administrators made a common mistake: granting that service account administrative permission to log into regular employee desktops.
* **What I did:** I ran a "Credential Spray" using NetExec to test the stolen password across every machine. It passed perfectly. 
* **The Breakthrough:** I launched a tool called **Evil-WinRM** from Kali. It connected over the network management wires, bypassed the workstation firewalls, and opened an interactive remote command-line terminal directly inside `HR-DESKTOP`. The attacker was officially in control of the employee's computer.

---

## 🕵️‍♂️ The Threat Hunt: How I Caught the Hacker in the Dark

Once the attack was finished, I swapped hats and became the **Blue Team Cyber Security Analyst**. I logged into **Splunk** completely blind—pretending I didn't know the hacker's IP address, the stolen usernames, or what commands were typed. 

I wrote advanced search queries based purely on **behavioral anomalies** to catch the threat:

### 🔬 Hunt 1: Isolating the Cryptographic Downgrade
To make passwords easy to crack offline, automated hacking tools force the Domain Controller to encrypt tickets using an ancient, weak method from the early 2000s called **RC4 (coded as 0x17)**. Modern Windows systems *never* use this naturally; they use secure AES encryption.
* **The Splunk Query:** `index=endpoint host=ADDC01 EventCode=4769 "0x17"`
* **The Catch:** This immediately stripped away the hacker's anonymity. Splunk surfaced a high-severity alert showing that the user account `svc-sql` was targeted using the weak `0x17` protocol, originating from the attacker's IP: `192.168.10.250`.

### 🔬 Hunt 2: Filtering Out the Corporate Noise
When you look at network logons, computers talk to each other automatically billions of times a second. This creates a mountain of background noise. In Windows, automated machine accounts always end with a dollar sign (like `HR-DESKTOP$`). 
* **The Splunk Query:** `index=endpoint EventCode=4624 Logon_Type=3 NOT (Account_Name="*$")`
* **The Logic Explained:** I ordered Splunk to show me **EventCode 4624** (Successful Login) and **Logon_Type 3** (connecting over network wires instead of sitting at the physical keyboard), but added the `NOT "*$"` rule to instantly throw away all automated computer noise.
* **The Catch:** The search instantly dropped over 200 spam logs down to a handful of clean lines, instantly exposing that the stolen `svc-sql` account had breached our workstations remotely from the outside network.

---

## 🎯 Key Takeaways & Career Skills Gained
By building and auditing this entire lifecycle from scratch, I mastered critical enterprise cybersecurity concepts:
* **Identity Threat Detection & Response (ITDR):** Understanding how attackers abuse legitimate user permissions to bypass traditional antivirus blocks.
* **The Principle of Least Privilege:** Why service accounts must be restricted to their dedicated servers and never granted administrative rights over standard user endpoints.
* **SIEM Data Engineering:** Learning how to clean, filter, and parse raw Windows Event telemetry using advanced Splunk Processing Language (SPL) to build production-grade detection logic.

---

## 🏁 Conclusion

This project successfully demonstrates the entire lifecycle of an identity-driven cyberattack, moving fluidly from initial network discovery to full network-wide lateral movement and defensive remediation. By executing this lab, I successfully validated how a single low-privileged network foothold can be leveraged by an adversary to harvest high-value service tickets, crack critical corporate credentials offline, and pivot deeply into standard employee workstations.

The core revelation of this project belongs to the **Blue Team defense phase**. By shifting into the role of a blind Threat Hunter, I proved that while attackers can hide behind valid user names and stolen passwords, they cannot hide their **cryptographic behavior**. 

Through the intentional engineering of Advanced Windows Audit Policies, I successfully forced the environment to report subtle indicators of compromise—specifically isolating old, crackable encryption downgrades (`0x17`) and filtering away thousands of lines of background computer account noise using advanced Splunk Processing Language (`NOT Account_Name="*$"`). 

Ultimately, this project highlights a critical industry truth: **preventing an intrusion is an IT function, but hunting down and exposing an active threat is the core job of a Detection Engineer.** This lab stands as definitive proof of my capability to architect enterprise security sandboxes, diagnose systemic operating system logging visibility gaps, and translate complex network behaviors into clean, automated security alerts that keep organizations safe from modern corporate breaches.

