# Integrating a SIEM

Before I dive in into implementing a SIEM:

<img src="../.gitbook/assets/Pasted image 2026-11-25 16.55.50.png" alt="" height="481" width="624">

This is our current (very scuffed) network setup. I was thinking of trying to clean this up by trying to put our OPNsense in front of all of our clients and computers, and not just my .50 subnet, but then I realized, that’s a lot of unnecessary work I’ll have to do for others.<br>

Essentially, if you look at our vmbr0 interface:<br>

<img src="../.gitbook/assets/Pasted image 2026-11-25 16.56.29.png" alt="" height="73" width="624">

We see that it has a DG of 192.168.1.1. This is the static IP address of the interface of our actual router, DD-WRT. (or Proxmox host’s gateway)  This means that all of our computers at least in our .1 subnet are just directly talking to the DD-WRT. This isn’t really optimal in an actual real environment because first of all, there’s no isolation or filtering. There isn’t a firewall between. Additionally having it on the same subnet makes everything all just messily condensed into one subnet, instead of having it separate like how .50 subnets should be client computers, .60 should be servers, and the .1 subnet could just broadly cover between the interface of vmbr0 and DD-WRT to create more separation.

Additionally, this also means that if any of the client computers want to talk to the .50 subnet, it wouldn’t be possible if there wasn’t a route in our DD-WRT that would tell any directions towards the .50 subnet needing to go back in the internal network instead of dropping out in nowhere.&#x20;

I can configure a route in our DD-WRT to do that, but I can’t even log in at the moment. (someone forgot the password). Additionally, it’s even more work asking everyone to add that route to their client/AD DCs locally. So what I think would be best, is by using the Linux Server I installed before, I put it on the vmbr0 switch so it’s on the .1 subnet and configure a static IP.&#x20;

### What is Elastic?

“Elastic SIEM (now part of the broader Elastic Security platform) is a free and open Security Information and Event Management (SIEM) solution built directly on top of the Elastic Stack”

The Elastic Stack composes of 5 elements:

* Elasticsearch: Database that searches and stores every log
* Kibana: Web Interface that includes dashboards, security apps, and searches, connects to Elastic Search&#x20;
* Elastic Agent/Beats: Installed on client devices and sends logs&#x20;
* Fleet Server: Manages/set rules all of the Agents in Kibana&#x20;
* Logstash: Optional data processing pipeline used to parse, clean, and enrich complex logs

Each component is separate and can be used separately. Elastic SIEM is a SIEM built on the ELK Stack.&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.14.39.png" alt="" height="168" width="624">

Check if we can even ping other computers first!&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 16.58.02.png" alt="" height="350" width="471">

Then ran commands:

Sudo apt update

sudo apt install default-jre default-jdk -y

Then I realized I don’t want to type all of these commands so I’ll just SSH:

<img src="../.gitbook/assets/Pasted image 2026-11-25 16.58.54.png" alt="" height="549" width="624">

{% code overflow="wrap" %}
```
sudo apt update && sudo apt install -y openssh-server
sudo systemctl enable --now ssh
wget -qO — https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg — dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo “deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main” | sudo tee /etc/apt/sources.list.d/elastic-9.x.list
```
{% endcode %}

The commands continue and continue but I basically just followed this guide:\
\
[https://medium.com/@justinsanthiyagu/installing-the-elk-stack-on-ubuntu-server-d823769b9e3e](https://medium.com/@justinsanthiyagu/installing-the-elk-stack-on-ubuntu-server-d823769b9e3e)<br>

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.07.53.png" alt="" height="585" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.07.54.png" alt="" height="225" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.07.55.png" alt="" height="268" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.07.56.png" alt="" height="188" width="485">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.08.31.png" alt="" height="441" width="624">

I then realized you also need to configure boring encryption keys

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.08.22.png" alt="" height="92" width="624">

Then I can actually install the blocks that make up of the SIEM.

{% code overflow="wrap" %}
```
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-9.5.5-linux-arm64.tar.gz
tar xzvf elastic-agent-9.5.5-linux-arm64.tar.gz
cd elastic-agent-9.5.5-linux-arm64
sudo ./elastic-agent install \
  --fleet-server-es=http://192.168.1.119:9200 \
  --fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE3OTI5ODc2MzcxODk6eWJFZHdiaWlScnEtUzdoSEppakQ1QQ \
  --fleet-server-policy=fleet-server-policy \
  --fleet-server-es-ca-trusted-fingerprint=78834C5C17CFE5F770D4A1A6C84F9BC440862FF1F7F3D887FFDFC608C71690AC \
  --fleet-server-port=8220 \
  --install-servers

sudo ./elastic-agent install --fleet-server-es=https://192.168.1.119:9200 --fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE3OTI5ODc2MzcxODk6eWJFZHdiaWlScnEtUzdoSEppakQ1QQ --fleet-server-policy=fleet-server-policy --fleet-server-es-ca-trusted-fingerprint=78834C5C17CFE5F770D4A1A6C84F9BC440862FF1F7F3D887FFDFC608C71690AC --fleet-server-port=8220 --install-servers
```
{% endcode %}

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.09.57.png" alt="" height="339" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.09.58.png" alt="" height="428" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.13.00.png" alt="" height="417" width="624">

Now we’re ready to add Elastic Agent on our windows endpoints.

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.13.01.png" alt="" height="433" width="624">

From here, there's quite a bit to do remaining to actually have working logs flowing through our SIEM.

* Set up the Linux Server as a static IP, preferably outside of the DHCP range our DD-WRT has.
* Configuring SSL on the server so HTTPs can run.&#x20;
* Set up Elastic Agent on my host computers&#x20;
* Configure syslog/sensor environment from our DD-WRT or OPNSense&#x20;
* Upgrade amount of storage on our Linux Server to Run nessus scan and bloodhound (possibly creating a new Kali Machine)

### Starting up the Linux Server&#x20;

From the time I wrote this section, I took a small break, and some things happened. I believe my firewall that I set up in OPNSense got completely wiped and restarted because some other teammate had issues censoring their SIEM. In addition, our entire server right now is just full of nonsense and VMs and I’m pretty sure our networking right now is just in the mud.

So to be honest, I’m just going to configure everything in the .1 subnet.  But first, is to set our server back and running:

{% code overflow="wrap" %}
```
df -h /                              
sudo systemctl status elasticsearch --no-pager
sudo systemctl status kibana --no-pager
sudo elastic-agent status
```
{% endcode %}

Doing this didn’t work because the memory wasn’t sufficient in my VM

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.17.42.png" alt="" height="288" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.17.59.png" alt="" height="316" width="624">

This then means I have to update the memory and mount more storage (for a future nessus scan)  After, adding more memory:

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.18.08.png" alt="" height="112" width="624">

I also needed to change the IP back to 119 since I kept getting errors, as DHCP gave my server a IP address of 192.168.1.118, which contradicted what I put manually before

{% code overflow="wrap" %}
```
sudo chmod 600 /etc/netplan/00-installer-config.yaml
sudo netplan apply
hostname -I
```
{% endcode %}

After I did this, I could finally log in back to ELK, until I got another error: All of my status health checks kept coming back with an EOF. I then checked the settings within fleet, and the “http” was supposed to be “https”&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.19.31.png" alt="" height="441" width="624">

### &#x20;Setting up Elastic Agent + Installing Sysmon&#x20;

Sysmon: System Monitor (Sysmon) is a free Windows system service and device driver from Microsoft's Sysinternals suite. Once installed, it runs as a protected background service across reboots to log deep system activity directly into the Windows Event Log

* Sysmon: Captures high-granularity behavioral telemetry - such as full process command lines, executable hashes, parent process IDs, network connections initiated by specific programs, registry modifications, and file timestamp changes. Events are written directly into the Windows Event Log in a structured format.

Syslog: Handles standard system and network event logging using a defined message structure containing a priority, timestamp, and hostname. It is often transmitted over UDP or TCP with low overhead, though it typically requires parsing or a relay (like rsyslog) to structure unstructured text for deep threat hunting\
\
Elastic Fleet is a centralized management system inside Kibana for configuring, monitoring, and upgrading Elastic Agents across hosts, hence we would need to add agents to a fleet.

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.20.29.png" alt="" height="427" width="584">

I needed to run this command to install Elastic Agent on my hosts:

{% code overflow="wrap" %}
```
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-9.5.6-windows-x86_64.zip -OutFile elastic-agent-9.5.6-windows-x86_64.zip
Expand-Archive .\elastic-agent-9.5.6-windows-x86_64.zip -DestinationPath .
cd elastic-agent-9.5.6-windows-x86_64
.\elastic-agent.exe install --url=https://192.168.1.119:8220 --enrollment-token=R2VHNGRxRUJtVThNeVJoMFJrM2s6bzVzWFhELVJJYlcyLXJXa2UwUi1NZw== --insecure
```
{% endcode %}

Before moving on, since my AD DC is on a different subnet and someone completely changed all of my router settings, the .50 subnet is no longer in contact with anything. Hence, I’ll just change it back to the .1 subnet.

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.22.11.png" alt="" height="440" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.22.12.png" alt="" height="321" width="395">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.22.13.png" alt="" height="496" width="583">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.22.14.png" alt="" height="179" width="624">

Changing it back to the virtual switch that deals with the .1 computers.

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.22.15.png" alt="" height="201" width="501">

Afterthen  using the powershell to PAINFULLY install the Elastic Agent:

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.22.15 (1).png" alt="" height="436" width="624">

Then on the same computer, we install Sysmon:

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.25.16.png" alt="" height="377" width="624">

{% code overflow="wrap" %}
```
cd C:\Windows\Temp
Invoke-WebRequest https://download.sysinternals.com/files/Sysmon.zip -OutFile Sysmon.zip
Expand-Archive Sysmon.zip -Force
Invoke-WebRequest https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml -OutFile sysmonconfig.xml
.\Sysmon\Sysmon64.exe -accepteula -i sysmonconfig.xml
```
{% endcode %}

Great it’s working - now doing the same for the Windows B Machine.

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.27.10.png" alt="" height="420" width="624">

Tried downloading to see that my windows machine ran out of storage… So now I need to add more&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.27.11.png" alt="" height="284" width="624">

Then going to Disk Manager&#x20;

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.27.12.png" alt="" height="511" width="624">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.27.13.png" alt="" height="308" width="540">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.27.15.png" alt="" height="285" width="489">

<img src="../.gitbook/assets/Pasted image 2026-11-25 17.27.16.png" alt="" height="396" width="624">

We repeat this for all of the VMs I have, but I'm feeling too tired to do it for all of them (and automating this would take a lot of work in regards to scripting)
