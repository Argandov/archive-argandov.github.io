---
title: "Compartmentalization Virtual Machine setup"
date: 2021-04-02T21:06:05-06:00
draft: false
tags: ["cybersecurity"]
toc: true # Table of contents
lead: "Quick guide to get started with testing virtual labs"

---

*Important: This following guide assumes your activities will be low-risk, so the isolation is kept at some degree lightly. If you have some advanced and specific needs, this guide may come short and you may need to do some more research.* <!--more-->*In this case, we would need to take some extra steps to fully isolate our environment, and this post would become very extensive, so keep that in mind.*

There are tons of tools available to effectively compartmentalize our virtual machines (From Type 1 or "bare metal" like [VMware's ESXi](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-B2F01BF5-078A-4C7E-B505-5DFFED0B8C38.html) or even [Qubes OS](https://www.qubes-os.org/) to Type 2 or "hosted" like [virtual Box](https://www.virtualbox.org/wiki/Downloads) or [VMware](https://www.vmware.com/mx/products/workstation-player.html)). We will use in this case the easiest to setup, which is Type 2, or "hosted".

For this example we will be using a Windows 10 as our "Host", and setup a Kali Linux guest for our testing. I chose Kali Linux as our demo, because it is readily packed with tools like wireshark and tcpdump, objdump, gdb, etc. which are good for pretty basic examination. But if you plan to setup a full Reverse Engineering environment, it is recommended to use [REMnux](https://remnux.org/). For Windows reverse engineer and malware analysis I have used [Flare VM](https://www.fireeye.com/blog/threat-research/2017/07/flare-vm-the-windows-malware.html), which is already packed with fantastic examination tools. It even has ***choco*** and ***Vim*** in its toolbox.

![image](https://geek-university.com/wp-content/images/oracle-virtualbox/type-2-hypervisor.jpg)
*Our Virtual Machine will "sit" in our Host machine, just like any other installed appliance.
Source: geek-university.com*

### Requirements for our home lab

- In this case, we'll use a Windows 10 Host (either 32-bit or 64-bit architecture is fine)
- 8+ GB memory
- Consider having around 50 - 100 GB of Hard disk space to allocate for the Virtual Machine(s), depending of course on your needs. Generally speaking, Windows guests will be more greedy than Linux, so always check your "Downloads" page for PC requirements every time you download a new ISO image.

### Setting up Virtualbox as our Hypervisor

1. We can get Virtual Box for free [here](https://www.virtualbox.org/wiki/Downloads), and install it. We will have our "guest" Operating Systems running just like another app/program on our system.

2. Download Kali Linux iso image [here](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/) [^1]

    *Be sure to check your host's architecture, whether it's a 64-bit or a 32-bit system by running `systeminfo` on your Windows' cmd.*

3. Open your Virtual Box, Click on *Machine* -> *New*, Name your new machine whatever way you'd like, choose the folder where you'd like your virtual machine to "sit" (I've even used external SSD and Thumbdrives for this, so it's pretty much up to you), choose *Linux* and *Debian* as the types, and move on to the next screen.

    Choose a Dynamically allocated virtual disk. I have never had any trouble with 10 GB testing Virtual Machines, but for real heavy lifting, 40-80 GB is more than enough for a Kali Linux VM. If you run out of space, it gets pretty tricky to expand your "virtual hard drive", and most likely you'd need to rely on LVM or "Logical Volume Management". (Unless you come up with an easier way to do it without wiping the whole thing and starting with a clean slate, let me know ;)

    Network Chuck has a pretty good video on virtual machines setup. [Watch it here](https://www.youtube.com/watch?v=wX75Z-4MEoM)

### Configuration before firing up our VM

- Open the ***Settings*** button on your new VM, and let's break down some useful things to know before starting:

- On the ***General*** tab, and under "Advanced", you can set or disable a bidirectional clipboard (Copying and pasting stuff between Host - Guest). [^2]

Under "Disk Encryption" you can set up a passphrase to achieve a VM encryption. Useful to know. You'd never want anyone to be able to retrieve your Cybersecurity/Pentesting labs in case your computer/hard drive gets stolen.

- Moving on to the ***system*** tab. Under "Motherboard" you will allocate some actual physical memory to the machine. Some people recommend 1-2 GB of memory for Kali Linux, but I've ran into some issues in the past, and for me it works just fine ith 4 GB.
Under "Processor", Allocate 1 or 2 cores to your Virtual Machine, and allow for 100% of Execution Cap.

- On ***Display*** tab: Under "Screen", 128 MB work just fine (We won't be running anything heavily graphical on it), Graphics Controller leave the "VMSVGA" without Enabling 3D Acceleration.

- On ***Network*** tab, we will need to decide how this will work for our specific needs. For security purposes, Virtualbox installs a "Virtual Network Interface" on the Host system, so it is capable of routing all the traffic via an internal NAT, which is good for isolation. For light, everyday use (And be able to communicate with the same network your Host is on) we can set it to "Bridged adapter" and select your Host's Network Interface Card (NIC).

To have an extra layer of protection, use ***NAT Network***. To set it up, go to the Virtual Box ***file*** tab -> ***preferences*** -> ***Network***, click "Add new NAT Network", choose your subnet/CIDR, and this will be your guest's Network for your chosen Network Interface under the Guest's settings.

For more information on Networking inside Virtual Box and what the different configurations do, you can [read this](https://www.virtualbox.org/manual/ch06.html)

- ***USB*** and ***Shared Folders*** enabled if needed. Don't enable them for a good compartmentalized environment. [^2]

### Fire up your VM

- Next, start your new Guest OS, and it won't have any disk image to run, so it will prompt you for one. Choose the .iso you previously downloaded, click "next", and you should be seeing Kali Linux firing up!

Most likely you won't have a full-screen machine. You can fix that here: [Link to kali.org](https://www.kali.org/docs/virtualization/install-virtualbox-guest-additions/)

##### A couple of useful keyboard shortcuts to know when inside a VM:

- Host (right Ctrl key) + F:    Fullscreen on/off
- Host + M:                     Minimize Guest's window

[^1]: Kali Linux can help at some degree with security, but for heavier security needs, we shouldn't rely on it.
[^2]: Most likely you won't be doing anything risky to your host or even your network in this lab, but this is important to consider for future Reverse Engineering and Malware Analysis Labs.These options are not recommended for achieving a theoretical full compartmentalization / Isolation. Consider what the ***risks*** are in your case, and what you will be doing with your VM before allowing some ***communication*** between Host / Guest.
