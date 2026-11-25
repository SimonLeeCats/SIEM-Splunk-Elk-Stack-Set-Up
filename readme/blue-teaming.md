# Blue-Teaming

Now we're doing the opposite of attacking, we're looking at the logs generated through our SIEM from the activity we did in the last segment.&#x20;

Regarding IOCs we would want to look for:\
\
**1. Network recon (the nmap scans)**

Mass connection attempts from your Kali IP (192.168.1.112) to .12, .111, .119

**2. SMBGhost exploit attempt**

SMB traffic to .12:445 from Kali

Possible SMB driver errors / crashes in the DC’s System log if the PoC destabilized it

Sysmon/network events showing the connection from 192.168.1.112

**3. Credential-based access attempts (WMIExec, the assumed creds)**

Event ID 4625 (failed logon)

Event ID 4624 (successful logon) with Logon Type 3 (network) if any landed

**4. DCSync / credential dumping attempts (bloodyAD, secretsdump)**

Event ID 4662 GUID (done in 3)&#x20;

**5. BloodHound/SharpHound collection**

Heavy LDAP queries to .12:389 from one host in a short window

**6. Sysmon Event ID 1 (process creation) D**

Event ID 4688

.lnk spawning powershell&#x20;

PowerShell Script Block Logging (4104)&#x20;

### Creating Kibana Visuals&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.09.40.png" alt="" height="455" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.09.42.png" alt="" height="592" width="579">

(the middle bar is our AD DC)

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.09.43.png" alt="" height="616" width="624">

Anyways, we need to go to Kibana Discover and sort by the agent name, and the time frame

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.09.44.png" alt="" height="456" width="624">

### 1. Network recon (the nmap scans)

We can just check the graphs/source ips.

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.13.33.png" alt="" height="401" width="624">

We only see 49. These are all mostly authentication attempts coming from the same IP address.&#x20;

However, it seems like there were no results from the nmap probes. I’m guessing it’s because Windows doesn’t log closed ports and not having a network sensor that was configured.

### &#x20;2. SMBGhost exploit attempt

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.14.18.png" alt="" height="589" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.14.20.png" alt="" height="129" width="624">

I tried this one initially because I didn’t know how KQL worked. This didn’t work because anding the names at once will always pretty much return False.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.14.24.png" alt="" height="195" width="624">

Windows Event 4625 records every failed authenticated attempt&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.14.29.png" alt="" height="376" width="624">

We see 23 failed attempts between November 5 and November 6.

The first failed attempt was on Nov 5 2026 20:29:54.

The spike of log in attempts was on Nov 5 2026 23:52 - 23:53. This came from 192.168.1.119 (our Linux server). It isn’t too alarming since we would inherently trust the Linux server since its from our internal network. Additional log in attempts were on Nov 6 2026 1:28:09. This came from the IP 192.168.1.112, which we know is our Kali Linux machine.<br>

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.14.39.png" alt="" height="413" width="372">

Logon Type 3 also means its been enumerated over the network, and its likely SMB. This is where I’m guessing I did guest users enumeration with smb. The “sers” is a truncated version of users, which hints that the attacker was guessing names. Now we would want to sort by IP address&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.14.44.png" alt="" height="279" width="624">

We don’t find anything, which makes sense since all of the attempts essentially failed&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.14.51.png" alt="" height="377" width="624">

Event ID 3

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.14.59.png" alt="" height="157" width="624">

Event 1001

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.15.04.png" alt="" height="488" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.15.12.png" alt="" height="437" width="624">

I think this should tell enough, but essentially there’s been attempts to authenticate via SMB via guess/null sessions, and VM trying to establish a connection (no 445 destination port were found)

### &#x20;3. Credential-based access attempts (WMIExec, the assumed creds)

What we can first do is just sort through the event logs 4625

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.18.19.png" alt="" height="377" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.18.22.png" alt="" height="216" width="624">

The LogonType 2 was probably me just entering the wrong password on the login page. (Since LogonType2 is interactive). But other than that, we essentially see a  Log4Shell (CVE-2021-44228) injection attempt by Nessus. This was Nessus testing every parameter whether they would actually resolve the domain.

{% hint style="info" %}
The mechanics: if any app logs that username through a vulnerable Log4j library, the ${jndi:ldap://...} makes the server reach out to the attacker’s LDAP URL and fetch/execute remote code. In the context of the lab, maybe the attacker ran Nessus scans or our blue team did it.
{% endhint %}

For Event Id 4624 (Successful Logon)

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.18.23.png" alt="" height="289" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.18.24.png" alt="" height="92" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.18.24 (1).png" alt="" height="387" width="624">

Essentially the attacker was able to authenticate over SMB shares over an anonymous session, which is in a sense a possible vulnerability, and should be fixed with a solution such as a GPO. This is likely through our rpcclient commands

I tried more event IDs such as 4661, 5140/5145, and 4662 and none of the results came back. I’m guessing this is because I did rpcclient, and l was able to initiate a null session, but wasn’t allowed to enumerate anything.&#x20;

### 5. BloodHound/SharpHound collection

Heavy LDAP queries to .12:389 from one host in a short window

For this one we’ll be looking into Windows Event Logs “Windows Event ID 4662 logs when an operation is performed on an Active Directory (AD) object.” Essentially, this is what BloodHound does to enumerate all of the users across the domain, so if we see a burst or spike in operations then we can assume it’s BloodHound at least in this lab.

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.18.26.png" alt="" height="346" width="457">

However, To view LDAP queries in the Windows Event Log, you must enable 4662 LDAP auditing. This is by default, turned off on DCs. This is because  Active Directory generates high log volumes during routine LDAP queries and background operations, and Windows requires two separate layers of configuration before it will record Event ID 4662. So we likely won’t get any events regarding Bloodhound.&#x20;

### 6. Sysmon Event ID 1 (process creation) <br>

I checked Event ID 4688 (new process created) and there was nothing. PowerShell Script Block Logging (4104):

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.18.28.png" alt="" height="307" width="624">

However, all of the script blocks were null:

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.18.29.png" alt="" height="551" width="248">

I don’t think there’s too much more but let’s create a dashboard to end it off and a timeline&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.18.29 (1).png" alt="" height="403" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.18.31.png" alt="" height="325" width="624">

There should be windows event 4662 from secretdump, but I couldn’t find any in m SIEM.

### Timeline

| Time (UTC)              | Source IP     | Event                                      | Indicator / Detail                                                                    | Attribution              |
| ----------------------- | ------------- | ------------------------------------------ | ------------------------------------------------------------------------------------- | ------------------------ |
| Nov 5, 2026 20:29       | 127.0.0.1     | Local admin logon (4624, Type 2)           | Interactive console logon as Administrator                                            | Benign (analyst)         |
| Nov 5, 2026 23:45       | 192.168.1.119 | Log4Shell injection (4625)                 | Username field contained ${jndi:ldap://...nessus.org/nessus} JNDI payload             | Nessus scan (authorized) |
| Nov 5, 2026 23:52-23:53 | 192.168.1.119 | Machine-account auth probes (Type 3)       | Repeated $ account logons against DC                                                  | Nessus scan              |
| Nov 6, 2026 01:00-01:16 | 192.168.1.112 | ANONYMOUS LOGON burst (Type 3)             | \~15 min of null-session authentication; automated enumeration                        | Red team (Kali)          |
| Nov 6, 2026 01:02-01:28 | 192.168.1.112 | Failed logons / credential guessing (4625) | Target users guest, users; SubStatus 0xC0000064 (no such user), 0xC0000072 (disabled) | Red team (Kali)          |

### Implement protective GPO's at DC level<br>

We first check what computer are connected to our domain in the AD DC, but I forgot my password and I didn't write it down so I need to reset it (oops). So we essentially need to hack on to it...&#x20;

Instead of booting up storage, I booted up the ISO file so I could run commands during the setup.

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.23.07.png" alt="" height="292" width="624">

After opening command prompt in the installation page, we can run some commands.

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.23.07 (1).png" alt="" height="251" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.23.08.png" alt="" height="149" width="624">

What this does is that we first rename the utilman.exe into utilman.exe.bak (a backup)

{% hint style="info" %}
“Utilman.exe (Utility Manager) is a built-in Windows accessibility tool located in C:\Windows\System32. It lets users open features like Magnifier, Narrator, and the On-Screen Keyboard including before logging into Windows.” It’s preinstalled and it ships with every Windows. Essentially, we’re replacing the real one with cmd.exe.
{% endhint %}

Then we copy cmd.exe and name the copy utilman.exe so now “utilman.exe” is actually a command prompt. Utilman runs as SYSTEM (highest privilege) and is reachable from the login screen without logging in (since accessibility tools must work before anyone signs in) Hence, by swapping it for cmd.exe, clicking Ease of Access at the lock screen opens a SYSTEM-level command prompt, from which you can reset any password:

After changing the boot order back to normal and rebooting it, we can  then use the Ease of Access tool.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.23.09.png" alt="" height="451" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.23.10.png" alt="" height="125" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.23.10 (1).png" alt="" height="547" width="607">

### Adding Linux Server to Domain

It's a common notion that you can't add non-windows computers to an AD Domain. This is false, as almost any computer can support joining a domain, as long as you install the tools. It's just that Windows OS natively has these features.&#x20;

We don’t see our Linux Server so let’s just add it. We’ll need to use realmd and sssd.\
\
realmd is a helper that discovers an AD/Kerberos domain and handles the enrollment handshake (like Kerboros) You use it once to join.&#x20;

SSSD (System Security Services Daemon) is the ongoing auth layer. After joining, SSSD is what actually:

* Talks to the DC to authenticate AD users when they log in
* Looks up AD users and groups (so id simon@rushhour.local works)
* Caches credentials so you can still log in if the DC is briefly unreachable

Since services such as LDAP/Kerberos doesn’t exist natively on Linux OS, we need to install translators that can do that

{% code overflow="wrap" %}
```
sudo apt update
sudo apt install -y realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
sudo realm discover rushhour.local
sudo realm join -U Administrator rushhour.local
sudo pam-auth-update --enable mkhomedir
```
{% endcode %}

I got an error with:

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.30.30.png" alt="" height="227" width="624">

First of all, a record is a direct map between a domain name and an IP address

A SRV record is a map between where the service is running. Essentially, it’s asking what machine is running, what service for the domain, and what port. An SRV record is a DNS entry that says: “Service X, using protocol Y, for domain Z, lives on host H at port P.” SRV records live side by side DNS A records, and \_msdcs is just the folder in DNS where all these AD service records live.  Which means that when it is empty or locked (REFUSED), no machine can look up where the DCs are, and domain operations fail.

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.27.43.png" alt="" height="79" width="624">

Ok apparently this isn’t working so I give up! We don't really need this but I thought it would've been more interesting.&#x20;

### Applying Domain-GPOs

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.31.36.png" alt="" height="433" width="624">

The only machine joined is our Windows VM, but it should still work. We apply/link the GPO to the OU of rushhour.local&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.31.38.png" alt="" height="465" width="624">

We created a GPO for the domain rushhour.local called Password Policy. This applies to every object, meaning every user and computer in that domain. Password policies specifically can only be applied to the domain level, not to an OU level.&#x20;

Now if we wanted to create a OU, then we can do so right in the GPM, but to actually move the users we need to go back toe ADUC. By default, our users sit in an container, not an OU. This means if we actually wanted to apply GPOs to a group, we needed to first create an OU.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.31.39.png" alt="" height="427" width="624">

Now lets apply a GPO to StandardUsers OU. It will:

* Disable SMBv1
* Enable Signing&#x20;
* Restrict Anonymous&#x20;
* Enable Logon/Logoff Logging

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.31.39 (1).png" alt="" height="455" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.31.40.png" alt="" height="484" width="624">

To apply it automatically (since its 90 minutes) we can just do:\
gpupdate /force gpresult /r <br>

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.31.42.png" alt="" height="172" width="425">

### Credential & Account Hardening / Remediate attack results&#x20;

To be honest there wasn’t much remediating for attack results considering the attack either needed credentials or for the victim to literally download an exe or some remote access tool, so there isn’t much “remediating attack results” in that.

However what we can do is make sure our accounts have least privilege, adding security groups, enforcing password policies, disabling LM Hashes.&#x20;

For adding security groups we do it in ADUC. The difference between security groups and adding GPOs is a security group is a list of users/computers that permissions get assigned to, ,and GPOs are more of a policy/configuration angle.

For example:

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.34.02.png" alt="" height="445" width="624">

Essentially, a security group has no privileges by itself. A group is just an empty container. It only grants whatever permissions you assign to it on resources. So a brand-new group like NonSubscription is already low privilege.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.31.43.png" alt="" height="435" width="624">

I then removed Domain Admins because Simon probably shouldn’t have Domain Admin. Now he’s not part of that security group that was delegated a lot of permissions.&#x20;

“Delegation in Active Directory (AD) is the practice of assigning specific administrative permissions to users or groups for defined tasks and scopes without granting full domain-wide or forest-wide admin rights””

Now for delegating, it’s about choosing what you want someone to manage for example, the user accounts inside an OU. If I add delegation under standard users OU, then I’m saying someone needs to control this OU.&#x20;

So I created a security group called Luck (probably shouldn’t place it under where it’s supposed to delegate the OU but whatever), that has full permission to edit and add users to StandDard Users. Now whoever is in the security group “Luck” can do that to the OU. Luck is a semi admin group.

Lesson basically: “in AD, you grant access by going to the resource (a folder, OU, or object) and adding the security group to its permission list,  never by opening the group and assigning powers to it, because a group is only the “who,” while the resource defines the “what.”

Regarding strengthening password policies, we can just do this in a GPO, but I like to keep my passwords unsafe (same thing with LM hashes)&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 18.31.44.png" alt="" height="287" width="624">

\
\
\
&#x20;

<br>
