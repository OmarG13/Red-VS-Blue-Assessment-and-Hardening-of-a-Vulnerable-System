# Red VS Blue
## Assessment and Hardening of a Vulnerable System

## _Network Topology_
  
![Network Topology](https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Network%20Topology.png)

## _Red Team Security Assessment_

### Reconnaissance 

![Recon](https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Recon1.PNG)

### Vulnerability Assessment

![Vulnerabilities](https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Vuln.PNG)

### Exploits

As an initial step, scans are run against the target machine to establish attack vectors.
```
Nmap -sV 192.168.1.105
```

![Nmap Scan1](https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Nmap1.PNG)

```
Nmap -Pn --script vuln 192.168.1.105
```

![Nmap Scan2](https://github.com/OmarG13/Red-VS-Blue-Assessment-and-Hardening-of-a-Vulnerable-System/blob/main/Images/Nmap2.PNG)
