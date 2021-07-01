---
title: "SSH Tunneling and Port-Forwarding guide"
date: 2021-05-06T21:06:05-06:00
draft: false
authorbox: false
tags: ["ssh"]
toc: true # Table of contents
lead: "A brief SSH tunneling & proxy methods guide"
thumbnail: /notebooks/img/ssh-thumbnail.PNG
---
Dynamic, Remote & Local port-forwarding. Tunneling and Hardening resources.
<!--more-->
### Remote port forwarding
<br>
SEND a port to a remote machine (Executed server-side):

![Remote port forwarding image](/notebooks/img/ssh-remote-port-forwarding.svg)

> ```ssh -R 8080:localhost:9999 user@server```
>
> Ports explained:
>
>Ssh [local port]:localhost:[Remote port] user@remote.server
>
>``` ssh -f -R 8080:local-ftp-server:21 user@192.168.1.10 -N```
>
>*(```-f``` runs in the background after connection; ```-N``` will not run any command remotely) Kill with ```pkill ssh```)*
>

It will "send" localhost's port 9999 to the remote server, through port 22 (ssh), to its local 8080 port. For example, use the browser to navigate to ```http://localhost:8080```; it will display whatever the remote server is "serving" in its local 9999 port.

- Examples of use:
	- The server is running an http server at p. 80, only accessible locally, or it is firewalled.
	- When an ftp service is allowed to be accessed only by certain hosts (Or only locally).

### Local port forwarding
<br>
GET, or "pull" a remote port (Executed client-side):

![Local port forwarding image](/notebooks/img/ssh-local-port-forwarding.svg)

> ```ssh -L 8080:localhost:9999 user@server```

Opens an 8080 port locally, and then "binds" it to the remote server's localhost 9999 port, through port 22 (ssh service). In short, remote port 9999 can be accessible locally at 8080.

- Another use example:
	- Same functionality. The only difference is which host is executing the command.
	- A port only accessible locally on another machine, for example an OpenVPN access http interface on the server, and we need to access it remotely, by "bringing" that port back for local use.

### Dynamic port forwarding through socksV5 proxy

![Dynamic port forwarding image](/notebooks/img/ssh-dynamic-port-forwarding.svg)

> ```ssh -D 6666 user@ssh-server```

This one has been called *"The poor man's VPN"*, and it will act just as a VPN would; It will proxy the connection through localhost's 6666 port to the remote server, tunneling all the traffic through the ssh protocol.

**Use case example:**

Once the connection is established, configure a browser to use a proxy, specifying localhost:6666. Then, all the traffic in that specific browser window will be tunneled through the ssh proxy.

##### For tunneling any specific application through our proxy:

We can use either proxychains (Already installed in distributions like Kali) or tsocks. For installing the latter in Debian:

> ```sudo apt-get install tsocks```

**Usage:**

> tsocks/proxychains nmap -Pn -sV -A 66.66.66.66
>
> tsocks/proxychains firefox
>
> tsocks/proxychains iceweasel argandov.github.io
>
> tsocks/proxychains curl ifconfig.io

** This can, of course, be used with whatever protocol we desire, not only SSH, but also Tor, Jondonym, etc.

### Port tunneling

Corkscrew - SSH through HTTP proxy - https://web.archive.org/web/20170510154150/http://agroman.net/corkscrew/

Sslh - Application protocol multiplexer (Server-side: Allow multiple services through a single port) https://github.com/yrutschle/sslh

### ssh security & hardening

> SSH SECURITY:
* [SSH hardening tutorial by Hackersploit (YouTube - 25 mins)](https://www.youtube.com/watch?v=Ryu3SDPYNb8)
* [Comprehensive SSH authentication and management of ssh keys (pdf by SANS institute)](https://www.sans.org/reading-room/whitepapers/authentication/architecture-configuration-hardened-ssh-keys-39940)
* [OpenSSH security guidelines by Mozilla Foundation](https://infosec.mozilla.org/guidelines/openssh.html)
