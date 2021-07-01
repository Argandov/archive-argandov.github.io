---
title: "Examine files in transit from pcap files in Linux"
date: 2021-04-02T21:06:05-06:00
draft: false
tags: ["cybersecurity","Linux"]
toc: true # Table of contents
lead: "Get Network Miner up and running in Linux in around 5 minutes"
thumbnail: /notebooks/img/pcap-Linux-thumbnail.PNG
---

In order for us to have a friendly interface for analizing network capture files (pcap), it's very useful to have monitoring tools like Network Miner, which lets us extract the data *in transit* as it happened at the time of the capture. It also allows us to "sniff" communications in real time but for this time we will only be using it for reading an already existing .pcap file.

What is Network Miner anyways? From the official site:

> *NetworkMiner is an open source Network Forensic Analysis Tool (NFAT) for Windows (but also works in Linux / Mac OS X / FreeBSD). NetworkMiner can be used as a passive network sniffer/packet capturing tool in order to detect operating systems, sessions, hostnames, open ports etc. without putting any traffic on the network. NetworkMiner can also parse PCAP files for off-line analysis and to regenerate/reassemble transmitted files and certificates from PCAP files.*

For this demo I was using a Debian distribution, and the process may be slightly different if you decide to use other distro, so the instructions for downloading and installing the required tools will be provided directly from the official pages.

First, since NetworkMiner is a Windows tool, we will need a useful .Net framework called "Mono", capable of running our program in our Linux OS. Maybe it's also possible to do this with "wine" if you already have it installed, but I am not that familiar with it. Let me know if this was the case and it worked for you.

##### Step 1: For installing mono

Download and install instructions for mono [here](https://www.mono-project.com/download/stable/#download-lin)

Follow the instructions and from here on the rest of the setup is pretty straightforward.

##### Step 2: Installing Network Miner

Download page for Network Miner [here](https://www.netresec.com/?page=networkminer)

Once the initial setup files are downloaded, we will need to unzip the contents, and run our new mono tool to execute the setup, and depending on your file paths, the command would look something like this:

```
$ unzip networkminer.zip
$ mono /path/networkminersetup.exe
```

And voilà, you should be having a windows setup screen for Network Miner installation. Follow up on the instructions and one we're done with that, let's just run NetworkMiner, again with mono:

```
$ mono /path/networkminer.exe /path/packetcapture.pcap
```
This last argument which is the path to the pcap file we want to open is optional, and we can also just run the program without opening any file just yet, and then manually open it using the GUI, it makes no difference.

For more information about the installation, direct from the official netresec site:

https://www.netresec.com/?page=Blog&month=2014-02&post=HowTo-install-NetworkMiner-in-Ubuntu-Fedora-and-Arch-Linux

Now we can start extracting valuable information from packet captures. And, since this tool was made by ***security professionals***, there are a couple options we can take advantage of, if we are doing forensics analysis work. There may be many more, but as an example, we can extract suspicious ***file hashes***, direct IP addresses/domains, digital certificates and even some credentials if it was the case at the time of the data capture.

Here we have excelent ***use samples for Network Miner from the official site:***

https://www.netresec.com/?page=Resources
