# Exploitation-and-Privilege-Escalation-on-Ubuntu-Vulnerable-Machine
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

<h1>Reconnaissance</h1> 
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

<h1>Initial Access</h1>
Since there's no direct exploit to the ssh version 7.2p2, the typical approach for SSH is:
- Find valid usernames 
- Brute force passwords
- Get a shell via legitimate SSH login 

Using the searchsploit command, I searched for exploits to target this version of SSH.
<pre>searchsploit openssh 7.2p2</pre>
<b>Username enumeration</b> is the chosen exploit as it is the only vulnerability matched with the SSH version.
<img></img>
Using the searchsploit command, I searched for exploits to target this version of SSH.
Username enumeration is the chosen exploit as it is the only vulnerability matched with the SSH version.
<pre>
  search ssh Username Enumeration
  use auxiliary/scanner/ssh/ssh_enumusers
</pre>
Then to use the required module;
<pre>use auxiliary/scanner/ssh/ssh_enumusers</pre>
<img></img>
The set RHOSTS is to the victim IP address.
<img></img>
The Username Enumeration module failed, even with using my own username word list which included the target’s username.
Username enumeration was the right choice, but exploits don't always work perfectly in real-world conditions. Moving to brute forcing is the next step. 
<pre>use auxiliary/scanner/ssh/ssh_login</pre>
This is the bruteforce module path used which attempts SSH login using a username/password list. If successful, it provides the password and creates a session between the victim and attacker machine.
<img></img>
<pre>set PASS_FILE /usr/share/wordlists/rockyou.txt</pre>
`/usr/share/wordlists/rockyou.txt` is the directory path to a password list, you can make your custom list for a faster bruteforce.

<pre>set user_as_pass true</pre>
This command tells metasploit to use the username also as the password, which is a common pattern and should be turned on by default but isn’t.
`run` the command module.

Metasploit successfully discovered valid login credentials for the SSH service: 
<b>Username: marlinspike</b>
<b>Password: marlinspike</b>
After the brute forcing was successful, a manual ssh login was done to gain remote access to the victim machine
<pre>ssh username@VICTIM_IP</pre>
<img></img>
For verification; `whoami`

#### Fun Fact
Here is a method to find the version of SSH port using metsploit. The result is the same as shown on the nmap scan result.
<pre>
<!-- Run this line by line -->
use auxiliary/scanner/ssh/ssh_version 
setg RHOSTS 192.168.1.150
run
</pre>

<h1>Privilege Escalation</h1>
To escalate my privilege I will be using
<pre>sudo -i</pre>
Alternatively, `sudo su` can be used as well.

<img></img>

I have successfully gained unauthoriazed access to another device through SSH. Note: SSH is only applicable for linux machines. For windows, RDP is the alternative. 





