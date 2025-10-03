<img width="733" height="268" alt="image" src="https://github.com/user-attachments/assets/cf6ba417-7ef4-47dc-89f9-b9bc605186ef" /># SIEM-Splunk-Elk-Stack-Set-Up

This is my first time doing a larger scale homelab so bare with me at some times. 

The first thing I did was to plan it all out in Cisco Packet Tracer. I planned the subnets, the device topology, and the overall strucuture of this home lab.
All external internet will be passed by pfSense (basically a firewall) and then processed by 2 hosts. The other are two servers, a Ubuntu Server for SIEM/Splunk, and a AD Windows server.
And the last one is actually a Kali Linux machine, hopefully used to do some pentesting or just general exploitation. 

<img width="769" height="691" alt="NetworkLayout" src="https://github.com/user-attachments/assets/bd4c0fbe-c44d-4599-8c87-2f2ca1c78791" />

The first thing I wanted to do (and argubaley) the hardest was to set up the devices itself in VMware. It was an actual pain to set all of these up to be working, so I'll just go one by one of how I setted or installed each one.

Endhosts

Ubuntu Desktop
So for this one I downloaded an ISO 

I won't havge 

Kali Linux
- I followed this tutorial on how to install Kali: https://www.youtube.com/watch?v=0J_Cxk4B-88 . It was simple enough, and I connected on "LAN Segment 1" in VMware.
<img width="733" height="268" alt="image" src="https://github.com/user-attachments/assets/81f70caa-23a0-40ac-a04e-9c13a2520b1a" />
<img width="733" height="269" alt="image" src="https://github.com/user-attachments/assets/5480799f-1b2a-46de-aa0b-cf324f8f87c8" />



Ubuntu SIEM
- I followed this tutorial: "https://www.youtube.com/watch?v=MztSvsbIHAg"
- the password is kali, username is server 



Then
