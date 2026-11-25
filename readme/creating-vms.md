# Creating VMs

Our goal for now is creating a Linux Server, OPNSense, and Windows VMs.

The first thing to do (and likely the most easiest) would be to install the Windows host VMs. Understanding the server view panel, I understood the pve (Proxmox virtual environment is our actual physical server. localnetwork (pve) is our default network zone. local (pve) is storage for files, such as ISO images, cand backups, and local-lvm (pve) storage for VM hard drives.

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.19.55.jpeg" alt="" height="288" width="552">

It was quite easy, and we just needed to put the iso file in Proxmox's storage.

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.20.16.png" alt="" height="365" width="624">

Once we did it (i needed to use a google drive link), we can then move on to actually installing the Windows VM.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.21.41.png" alt="" height="280" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.21.45.png" alt="" height="424" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.21.50.png" alt="" height="276" width="415">

{% hint style="info" %}
One thing that piqued my interest was the stuff being saved to /var/lib/vz, etc…

I researched it and tried to understand what this all really meant. Apparently, Proxmox similarly works like a Linux OS. Since Proxmox VE is built on Debian Linux it contains the file system of Linux.&#x20;



Understanding the path:\
/var is for variable, that holds data that changes when systems/applications run\
/var/lib is for data that programs keep permanently (for example after boot)\
After researching a bit /vz is a name for Proxmox’s folder (it came from OpenVZ)
{% endhint %}

Now for setting up a firewall/router, we can use OPNSense.&#x20;

{% hint style="info" %}
OPNsense is a free, open source firewall and routing platform based on FreeBSD. It filters traffic with stateful firewall rules and also handles NAT, VPNs, DHCP, DNS, and traffic monitoring, all from a web interface. You can run it on a mini PC, a server, or a virtual machine, and add features with plugins, such as Zenarmor for next-generation firewall features like application control and web filtering.
{% endhint %}

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.23.45.jpeg" alt="" height="468" width="624">

For OPNSense, it was pretty much the same process for deploying the Windows VM, essentially just uploading the ISO file, and running it.&#x20;

### More VMs

For me, I wanted to set up 2 more Windows Hosts, so it's pretty much the same process.

I named those two the following: WindowsPro10\_A\_Simon, WindowsPro10\_B\_Simon

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.31.59.png" alt="" height="52" width="260">

And same thing for a Linux server with a host name of linux-server-simon.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.32.18.png" alt="" height="329" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.34.41.png" alt="" height="341" width="537">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.34.49.png" alt="" height="331" width="624">

One thing I always wondered was why I needed to remove the installation medium to then continue with the process. I learnt online this comes down to boot order. Instead of the computer loading up the installer again, the computer loads up the newly installed OS. Similarly to taking out a USB or a CD.

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.35.01.png" alt="" height="279" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.35.14.png" alt="" height="405" width="624">
