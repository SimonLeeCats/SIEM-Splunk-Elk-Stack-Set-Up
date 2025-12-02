# SIEM-Splunk-Elk-Stack-Set-Up

This is my first time doing a larger scale homelab so bare with me at some times. TEST TEST TEST The first thing I did was to plan it all out in Cisco Packet Tracer. I planned the subnets, the device topology, and the overall strucuture of this home lab. All external internet will be passed by OPNSense (basically a firewall) and then processed by 2 hosts. The other are two servers, a Ubuntu Server for SIEM/Splunk, and a AD Windows server. And the last one is actually a Kali Linux machine, hopefully used to do some pentesting or just general exploitation.

<figure><img src="https://github.com/user-attachments/assets/befdd76f-04d4-4327-b86e-dd4c29cfcbec" alt=""><figcaption></figcaption></figure>

* The firewall is pfSense but i changed it to OPNSense since i struggled in making it work

Next, was basically the hardest part. It was to set up the devices itself in VMware. It was an actual pain to set all of these up to be working, but since it isn't too relevant I won't go too deep. For the endhosts, like Kali Linux, Ubunutu Desktop, and Windows 10, it was relatively straightfoward. However, the pfSesne (firewall) was an absolute nightmare to set up.

Kali Linux

* I followed this tutorial on how to install Kali: https://www.youtube.com/watch?v=0J\_Cxk4B-88 . It was simple enough, and I connected on "LAN Segment 1" in VMware.
* (Also if you're following along make sure to do this action) <img src="https://github.com/user-attachments/assets/81f70caa-23a0-40ac-a04e-9c13a2520b1a" alt="image" data-size="original">

![image](https://github.com/user-attachments/assets/5480799f-1b2a-46de-aa0b-cf324f8f87c8)

Ubuntu SIEM

* I followed this tutorial: "https://www.youtube.com/watch?v=MztSvsbIHAg"
* the password is kali, username is server

Windows Server 2022

* Password kali4321!

OPNSense ![image](https://github.com/user-attachments/assets/62143eb9-539d-4a90-982c-fb693adba4de)

* I put "1.1.1.1" as the name server

Then

Setting up Active Directory in Windows Server 2022

![image](https://github.com/user-attachments/assets/72f804f4-e962-414e-a4ff-dd4171707ef2) ![image](https://github.com/user-attachments/assets/41639e2c-05aa-446f-a5b0-a1440ffa0f2f) ![image](https://github.com/user-attachments/assets/8d166374-6547-4df3-821b-a3243ce5bba2) I gave the domain name of "server.org" with a DSRM password of kali4321! ![image](https://github.com/user-attachments/assets/454c4278-b20a-47d2-94b4-ce8b43f002c0) ![image](https://github.com/user-attachments/assets/e8741135-132d-4d1f-9629-ee6f7e8da73f) ![image](https://github.com/user-attachments/assets/fa4e2ac9-0a39-4f7a-9634-c81d168e1304)
