# Allowing internet access

After installing Proxmox, we still needed to install the actual virtual machines we'll be working on. However, first thing is making sure our Proxmox could actually communicate to the internet, and installing Remote Access tools so we can remote on to it virtually (instead of doing everything physically through a cable)

The issue with out server, is that we needed to connect our server to the internet, but we couldn't do so with an Ethernet port. And since we were only allowed to use the guest wifi, we could either use a wifi adapter, or an router. Luckily, routers were given, so we could use that router to translate WiFi into internet in our server.

<img src="../.gitbook/assets/Pasted image 2026-11-25 05.53.12.png" alt="" height="658" width="508">

We were given a DD-WRT Router. We connected it to an Ethernet port, and plugged it into our computers. We then went to the console for the router and did basic configurations.

<img src="../.gitbook/assets/Pasted image 2026-11-25 05.58.30.jpeg" alt="" height="288" width="564">

This included enabling DHCP, setting a static IP, and other configurations.

<img src="../.gitbook/assets/Pasted image 2026-11-25 06.00.54.jpeg" alt="" height="288" width="624">

After we configured the basic settings for our router, we then can move on to installing a remote access tool. Specifically, we'll be using Tailscale.&#x20;

### What is Tailscale

I didn’t understand what Tailscale initially was and how it worked. Rather “It is a mesh VPN that makes your own devices act like they are on one private network.” Essentially, Tailscale gathers all of the devices and puts them all in a private network (subnet) so all of the devices can directly talk to each other. It’s different from a normal VPN (like Proton VPN, NordVPN, etc…) in which they route your traffic through their servers to hide your IP or change location. Instead, the purpose being to hide your IP, a mesh vpn purpose is to get private access to machines and services you own.&#x20;

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

I also needed to understand what Wireguard was. At first I didn’t know VPN products had these like VPN protocols. I only understood that Wireguard was a protocol, similar to OpenVPN and IPsec. I learnt that Wireguard was a more standardized, advanced protocol in comparison to the older VPN protocols. It prioritized simplicity, strong encryption, and dynamic routing.&#x20;

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Additionally, I wasn’t sure of how all of the IPs worked and why everything had a prefix of 100. What I realized was that when I downloaded Tailscale, it created a Network Adapter that tunneled all connections that was 100.64.0.0/10.

So whenever I would enter a IP address that would fall into that subnet, it would redirect that to Tailscale and towards Proxmox.

I also researched on how Data Planes and Control Planes worked in Tailscale. Essentially, the Control Planes deal with all of the logic, and Data Planes deal with the actual sending of the data, much like how SD-WAN networking works. Then, Tailscale (Coordination server) assigns every device with a (permanent) IP address and puts them all into one private network.

Now in regards to downloading Tailscale on Proxmox:<br>

<img src="../.gitbook/assets/Pasted image 2026-11-25 07.09.52.jpeg" alt="" height="288" width="472">

We first of all needed to head over to the CLI of Proxmox. I'm guessing since Proxmox is meant to be used over an IP/Remotely, directly connecting to the server running it only gives us the CLI, and not any of the browser GUI. Anyways, running this command to download Tailscale:

curl -fsSL https://tailscale.com/install.sh | sh

Let's us run the Tailscale client on the server. With another command and authenticating to Tailscale (also creating an account):

tailscale up

We were able to log in remotely over the 8006 port.&#x20;

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

After downloading the script on the Proxmox shell, I would also need to attach a :8006 port as well. I didn’t initially understand why I needed to attach a port number - so I researched why. Apparently the port is the default HTTPS port used by Proxmox Virtual Environment (PVE) and Proxmox Mail Gateway (PMG) for their web management graphical user interface.&#x20;

{% hint style="info" %}
Port 8006 is the default HTTPS port for the Proxmox VE web management interface and API; access it at `https://<server-ip>:8006`
{% endhint %}

And this should be it in regards to setting up internet access!&#x20;
