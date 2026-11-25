# Setting up Active Directory

### Installing Local GPOs

Before creating an Windows Server to set up Active Directory, I wanted to first experiment created Local GPOs.&#x20;

{% hint style="info" %}
GPOs, or Group Policy Objects, are features in Microsoft Windows that allow centralized management and configuration of operating systems, applications, and user settings within an Active Directory environment. They enable administrators to enforce specific policies, such as security settings and software installations, across multiple computers and users in a network.
{% endhint %}

I wasn’t really familiar with putting GPOs on a OS that wasn’t Windows Server 2022, so I researched online what type of tools exist, and how implementing a GPO would actually work.

From what I learnt, Domain GPO(s) exists only in the sense of an AD DC with clients pulling the GPOs from them. If you’re trying to configure them locally, it is more technically just editing one big GPO for my local computer, and editing 5 policy settings specifically, not implementing GPOs for your network. If you want to "implement 5 GPOs” This would require an AD DC and endpoints that are connected to your domain/tree/forest whatever.&#x20;

**Local Tools:**

Local Group Policy Editor: Security settings for local PC\
Local Security Policy

**Domain tools:**

ADAC/ADUC (AD Administrative Center): Managing standard Active Directory objects like users, groups, computer

Group Policy Management: Create GPOs that apply to many PCs

PowerShell (AD module): Can also create GPOs and are the backbone of the tools listed here

For this part, I’ll be using the Local Group Policy Editor. But first of all, I tried to understand how like all the folders work and all here:

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.40.35.png" alt="" height="521" width="236">

At its core, A Group Policy Object (GPO) is a collection of group policy settings. An individual policy setting defines a specific configuration, such as a policy setting that prevents a user from accessing registry-editing tools. If you define that policy setting and then apply it to a user, that user will be unable to run tools such as Regedit.exe. (most settings are just registry changes)

The view tree is laid out into two categories, users and computers. But what's more interesting is that the two split into 3 more categories. (in Group Policy Management it splits into preferences and policies first), Software Settings, Windows Settings, and Administrative Templates. Administrative Templates is the bigger one, as it contains thousands of settings for Windows features: Control Panel, Desktop, PowerShell, Defender, etc…

**GPO1**

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.42.31.png" alt="" height="372" width="624">

Changed Account Threshold into 3 invalid login attempts. Avoid brute forcing &#x20;

**GPO2**

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.42.32.png" alt="" height="338" width="483">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.42.33.png" alt="" height="80" width="396">

Reading it, “LLMNR is a protocol that allows both IPv4 and IPv6 hosts to perform name resolution for hosts on the same local network without requiring a DNS server or DNS configuration.” This one essentially just stops attacks from spoofing domain names lookups.\
\
https://tcm-sec.com/llmnr-poisoning-and-how-to-prevent-it/

**GPO3**

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.42.33 (1).png" alt="" height="462" width="493">

Since Powershell is one of the common scripting languages nowadays for unethical purposes, enabling logging can be great!

**GPO4**

Administrative Templates > Windows Components > AutoPlay Policies > Turn off

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.42.34.png" alt="" height="79" width="474">

Essentially whenever you plug in a USB device, there’s a pop up requesting for permission or even sometimes just executing automatically. &#x20;

Disabling this  prevents a USB drive that potentially has malware from launching immediately, as on default this setting is enabled.&#x20;

**GPO5**

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.42.35.png" alt="" height="422" width="536">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.42.36.png" alt="" height="398" width="345">

This one is a little bit different. This is a policy setting, but specifically an audit policy setting. “Audit Logon” is one setting, and the Success and Failure checkboxes are its value

On default (especially on this client computer), it logs attempts on Success. However, it doesn't log Success and failures. Enabling both can help find potential bad actors.&#x20;

### Installing Window Servers

First of all, load an ISO file into Proxmox and download the OS. I first made sure to change the hostname and IP address (to a static instead of DHCP).&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.45.37.png" alt="" height="353" width="316">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.45.44.png" alt="" height="432" width="624">

After restarting my server (so the updates can be set), I then needed to assign this server into roles.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.46.20.png" alt="" height="461" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.46.32.png" alt="" height="117" width="288">

I created a forest called rushhour.local and ran some tests, such as pinging other computers and running powershell commands.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.47.12.png" alt="" height="293" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.47.29.png" alt="" height="49" width="624">

I made sure that all of the devices, were on the same virtual switch on Proxmox.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.48.02.png" alt="" height="181" width="470">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.48.08.png" alt="" height="128" width="442">

### Clarifying Misconceptions + Network Troubleshooting

A DC doesn’t have its own separate domain name. It belongs to the domain like any other member, so its FQDN is just hostname + domain name. That means that whenever a computer would want to communicate with the AD DC, it tries to find the FQDN through DNS

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.53.52.png" alt="" height="159" width="624">

First of all, A network adapter (also called a Network Interface Card or NIC) is the hardware component inside a computer or device that translates data into signals so it can communicate with a local network and the internet

Understanding what this meant helped me troubleshoot some problems. Essentially, net0 means the VM’s first network card, and followed by that means the e1000 is what network card model Proxmox pretends to be (an Intel card that Windows has drivers for). For every network card, I imagine it as one ethernet port in a real physical switch.&#x20;

So vtnet0 is simply OPNsense’s name for its first network card. vtnet1 is the second card, vtnet2 the third, and so on.

BC:24:11 is the network’s card MAC Address. The bridge is a virtual network switch that lives inside the Proxmox server. Everything plugged into the same switch can send traffic to each other directly.

However, what I needed to understand was that vmbr0 was like an actual switch (with the firewall rules being 1). A VM on vmbr0 and a VM on vmbr1 can’t talk to each other directly through Proxmox, just like two separate physical switches with no cable between them.&#x20;

Our actual real physical router (DD-WRT) is connected to our Proxmox server that has a network card called nic1 that is directly connected to our virtual switch vmbr0 that has pretty much infinite ports (since it's virtual). Which is why my Win10 client got 192.168.1.111 automatically: DD-WRT’s DHCP handed it out. Essentially, our Proxmox isn’t doing anything at the current moment. It just has an IP address from our actual router. Right now, all of our connections are through this one virtual switch, which is why all of the 192.168.1 / 24 computers can effectively communicate. &#x20;

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

However, if I want to place my computer on a different subnet, this basically means that instead of sending it directly to the switch, it sends it to the default gateway towards our router. For example, if my DC (192.168.50.4, gateway .254) was trying to reach the client (192.168.1.111), no device has that address of the gateway. The DC can’t reach the client, DD-WRT, or the internet. Meanwhile, if our client (192.168.1.111) is trying to reach the DC (192.168.50.4), it doesn’t know where it is, so according to vmbr0, its DG is 192.168.1.1. But again, our physical router DD WRT has no idea where the 50 subnet is, so it essentially dies. &#x20;

The first thing to do is to create your switch on Proxmox (vmbr2)

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.10.png" alt="" height="223" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.11.png" alt="" height="251" width="624">

Then, you would go to whoever has the …50 IP Address and assign it to the vmbr2 switch. For devices with a …1 IP address you would leave it on the vmbr0 address.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.12.png" alt="" height="240" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.13.png" alt="" height="443" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.14.png" alt="" height="243" width="624">

If I'm being honest, this is an awkward way of setting up the router, since that we really should be doing this on our actual, physical router instead of our virtual router, but other people had their VMs on the .1 subnet, and I didn't want to affect their routing.

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.14 (1).png" alt="" height="289" width="432">

Then going to OPNSense:

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.15.png" alt="" height="437" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.16.png" alt="" height="108" width="575">

Block bogon refers to the practice of filtering out network packets that originate from invalid or unassigned IP addresses, known as bogons. This is done to enhance network security by preventing unauthorized access and reducing potential malicious activities.

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.17.png" alt="" height="76" width="624">

And since I wasn't allowed to touch the router for now (because someone forgot the password for our router)&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.17 (1).png" alt="" height="417" width="571">

Finally, after all of this mess, I then asked Claude to generate a network diagram.

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.18.png" alt="" height="345" width="471">

Essentially: internet ← DD-WRT ← vmbr0 ← OPNsense ← vmbr2

And after a lot of testing, we were able to ping other computers in different subnets.

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.57.19.png" alt="" height="499" width="461">

This requires adding on every client device:

route -p add 192.168.50.0 mask 255.255.255.0 192.168.1.254

Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.50.4\
Instead of having routing on the clients we can do 2 different ways:\
We can either put the route on  the actual DD-WRT instead. In DD-WRT,  it's adding a static route: destination 192.168.50.0, mask 255.255.255.0, gateway 192.168.1.254, or having all of the clients behind this OPNsense firewall.&#x20;

### Joining a Windows Computer to Domain&#x20;

1. Check if connection is established (Test Pinging)
2. Use this command: Add-Computer -DomainName rushhour.local -Credential Administrator@rushhour.local -Restart
3. Put in credential

<img src="../.gitbook/assets/Pasted image 2026-11-25 08.22.33.png" alt="" height="393" width="624">

You can also do it through the GUI

<img src="../.gitbook/assets/Pasted image 2026-11-25 08.22.34.png" alt="" height="473" width="624">

After restart, create a user in the AD DC (Dashboard)

<img src="../.gitbook/assets/Pasted image 2026-11-25 08.22.35.png" alt="" height="480" width="624">

And then, I signed in (after adding credentials)

<img src="../.gitbook/assets/Pasted image 2026-11-25 08.22.37.png" alt="" height="249" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 08.22.47.png" alt="" height="219" width="454">

Then adding to Domain Admin

<img src="../.gitbook/assets/Pasted image 2026-11-25 08.22.52.png" alt="" height="431" width="624">

Since security groups are about granting permissions (file shares, admin rights, etc), I essentially added permissions to this user (Simon) under 2 security groups (Admin and Regular Domain user) which allows Simon to edit even the AD DCs.&#x20;





<br>
