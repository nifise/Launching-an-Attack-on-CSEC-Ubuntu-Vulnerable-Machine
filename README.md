# Launching-an-Attack-on-CSEC-Ubuntu-Vulnerable-Machine
### The purpose of this security assessment was to simulate how attackers can gain access to outdated or misconfigured systems and demonstrate the potential risks associated with vulnerable services.

This attack was performed within a virtual environment, where only the machines assigned within the CSEC Ubuntu vulnerable lab were targeted.
<b>No external systems were scanned or attacked.</b>
### Techniques used (MITRE ATT&CK)
<table>
  <tr>
    <th>ID</th>
    <th>Technique</th>
    <th>Description</th>
    <th>Tool Used</th>
  </tr>
  <tr>
    <td>T1595</td>
    <td>Active Scanning</td>
    <td>Acquisition of live hosts IPs and MAC addresses</td>
    <td>arp-scan</td>
  </tr>
  <tr>
    <td>T1595.002</td>
    <td>Active Scanning: Vulnerability Scanning</td>
    <td>Victim vulnerability scans e.g. outdated port services</td>
    <td>nmap</td>
  </tr>
    <tr>
    <td>T1190</td>
    <td>Exploit Public-Facing Application</td>
    <td>Exploiting outdated public facing services</td>
    <td>Metasploit</td>
  </tr>
</table>

<table>
  <tr>
    <th>Item</th>
    <th>Details</th>
  </tr>
  <tr>
    <td>Target</td>
    <td>CSEC Ubuntu velnerable machine</td>
  </tr>
  <tr>
    <td>Attacker Machine</td>
    <td>Kali Linux</td>
  </tr>
  <tr>
    <td>Tools used</td>
    <td>arp-scan, nmap, metasploit</td>
  </tr>
</table>

### Reconnaissance 
<pre>sudo arp-scan -l</pre>
I launched an arp sweep to gather a list of IP addresses in my LAN. 192.168.1.150 is the IP address of the victim machine. I used arp-scan because it was faster than netdiscover and i already knew the mac address of the csec machine.

<img></img>
<pre>nmap -sV -O --open 192.168.1.150</pre>
In this case I want only the open ports to be listed, the version of the services in the ports and the OS version. 
-sV - Displays version of the service
-O - Displays version of operating system
–open - Displays only open ports
The screenshot below shows the three open ports and their version as requested in the command. 
The chosen port will be port 22 with SSH. The OpenSSH version 7.2p2 is an outdated version with several known exploits. One notable vulnerability is CVE-2016-6210, which allows for username enumeration via timing attacks on the SSH daemon. 
<img></img>

#### Fun Fact
Here is a method to find the version of SSH port using metsploit. The result is the same as shown on the nmap scan result.
<pre>
<!-- Run this line by line -->
use auxiliary/scanner/ssh/ssh_version 
setg RHOSTS 192.168.1.150
run
</pre>

### Initial Access
Since there's no direct exploit to the ssh version 7.2p2, the typical approach for SSH is:
- Find valid usernames 
- Brute force passwords
- Get a shell via legitimate SSH login 

Using the searchsploit command, I searched for exploits to target this version of SSH.
<pre>searchsploit openssh 7.2p2</pre>
<b>Username enumeration</b> is the chosen exploit as it is the only vulnerability matched with the SSH version.

<pre>
  search ssh Username Enumeration
  use auxiliary/scanner/ssh/ssh_enumusers
</pre>









