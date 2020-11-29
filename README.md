# Red VS Blue
## Assessment and Hardening of a Vulnerable System

## _Network Topology_
  
<img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Network%20Topology.png>

## _Red Team Security Assessment_

### Reconnaissance 

<img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Recon1.PNG width=700>

### Vulnerability Assessment

<img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Vuln.PNG width=700>

### Exploits

As an initial step, scans are run against the target machine to establish attack vectors.
```
Nmap -sV 192.168.1.105
```

<img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Nmap1.PNG width=500>

```
Nmap -Pn --script vuln 192.168.1.105
```

<img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Nmap2.PNG width=500>

With Apache Httpd open and Webdav showing up on our scans, it was clear the server would be accessible via a browser.
Navigating to ```192.168.1.105``` allows us to browse through the server folders where there are multiple reference to a "secret_folder" directory. There is also a list of employee names (good source for user names) as well as roles and responsibilities. All very useful in planning our attack.

<img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Http1.PNG width=500>

Navigating to the secret_folder directory requires us to use the user name and password for Ashton.

<img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Http2.png>

Cracking Ashton's password is attempted using Hydra.

```
Hydra -l ashton -P /usr/share/wordlists/rockyou.txt -s 80 -f -vV 192.168.1.105 http-get /company_folders/sercret_folder
```
<img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Hydra.png>

<img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Http3.png>


✨✨ SUCCESS!! ✨✨
