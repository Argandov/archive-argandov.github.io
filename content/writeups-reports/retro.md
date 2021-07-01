---
title: "Tryhackme writeup: Retro"
date: 2020-05-16T21:06:05-06:00
draft: false
authorbox: false
toc: true
thumbnail: /writeups-reports/files/retro/retro-thumbnail.PNG
---

Boot to root: Retro wordpress theme & exploitation of CVE-2019-1388

<!--more-->

### Initial recon + Access:

**Results from nmap** (Filtering was enabled. The option -Pn was necessary in this box):

```
80/tcp   open  http
3389/tcp open  ms-wbt-server
```

**IIS Server running @ port 80**

We can run a directory scan with gobuster, but we can first try "retro" as the directory.

![retro-http](/writeups-reports/files/retro/retro-http-main.png)

**Wordpress**

Navigating within the page, we see a login page at /retro/wp-login.php

User: Wade, password unknown yet. Admin user was invalid at .../wp-login.php, so we guess "wade" can be an administrator of the site.

### Access

There is one blog post called "*Ready player one*", in which there is a comment from Wade himself, about trying not to forget his favourite character "parzival" and so on.

We try logging in with \<wade:parzival\>, and we're into the wordpress administration page! We could try to exploit wordpress themes by uploading a custom php reverse shell, but seeing there is an rdp service running, we can try to connect with xfreerdp and this credentials.

![retro-http](/writeups-reports/files/retro/retro-rdpconnection-usertxt.png)

And we're in!

### Privesc

Opening Google chrome (I was expecting to see some information on History/passwords/etc.), there is a bookmark for NIST referencing CVE-2019-1388, which is itself a clue (maybe), about a vulnerability issuing "Windows Certificate Dialog Elevation of Privilege Vulnerability".

### CVE 2019-1388

[(NIST) CVE-2019-1388](https://nvd.nist.gov/vuln/detail/CVE-2019-1388)

This points to an issue with the UAC in Windows:

> Whenever we run a verified executable (This can't be done with unsigned .exe) as **administrator**, and we're prompted for the admin password, we can click the link of the path to the certificate, and this will open a browser as administrator, since we're still on the UAC.
>
> Then, we can save the browser's page (*save as*), and at the saving window, instead of actually saving the html, we navigate to C:/users/user/windows/system32/ and open cmd.exe.

PoC video: https://www.youtube.com/watch?v=6kVVwBXOPW0

Remember, we're still under the UAC process, so whatever we do here, will be executed as **administrator**:

![retro-http](/writeups-reports/files/retro/retro-admin-shell.png)

This way we secured the flag by just reading it from the cmd *(Or opening ```> notepad root.txt.txt``` because the selection and copy/paste functions directly at the terminal were not available)*.
