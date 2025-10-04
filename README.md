# SIEM-Splunk-Elk-Stack-Set-Up

This is my first time doing a larger scale homelab so bare with me at some times. 

The first thing I did was to plan it all out in Cisco Packet Tracer. I planned the subnets, the device topology, and the overall strucuture of this home lab.
All external internet will be passed by pfSense (basically a firewall) and then processed by 2 hosts. The other are two servers, a Ubuntu Server for SIEM/Splunk, and a AD Windows server.
And the last one is actually a Kali Linux machine, hopefully used to do some pentesting or just general exploitation. 

<img width="769" height="691" alt="NetworkLayout" src="https://github.com/user-attachments/assets/bd4c0fbe-c44d-4599-8c87-2f2ca1c78791" />

Next, was basically the hardest part. It was to set up the devices itself in VMware. It was an actual pain to set all of these up to be working, but since it isn't too relevant I won't go too deep.
For the endhosts, like Kali Linux,  Ubunutu Desktop, and Windows 10, it was relatively straightfoward. However, the pfSesne (firewall) was an absolute nightmare to set up. 

Kali Linux
- I followed this tutorial on how to install Kali: https://www.youtube.com/watch?v=0J_Cxk4B-88 . It was simple enough, and I connected on "LAN Segment 1" in VMware.
- (Also if you're following along make sure to do this action) 
<img width="733" height="268" alt="image" src="https://github.com/user-attachments/assets/81f70caa-23a0-40ac-a04e-9c13a2520b1a" />
<img width="733" height="269" alt="image" src="https://github.com/user-attachments/assets/5480799f-1b2a-46de-aa0b-cf324f8f87c8" />

Ubuntu SIEM
- I followed this tutorial: "https://www.youtube.com/watch?v=MztSvsbIHAg"
- the password is kali, username is server

Windows Server 2022
- Password kali4321!



Then
