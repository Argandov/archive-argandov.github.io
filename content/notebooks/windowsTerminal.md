---
title: "Windows Terminal: Keeping a neat and clean workspace"
date: 2020-11-01T21:06:05-06:00
draft: false
authorbox: false

---

---

When it comes to open sessions, monitoring your network or communicate between servers/hosts, organization is key for an agile mind.

While there are lots of options to establish a connection between hosts (Like Termius, PuTTY, or even the original terminal), I found the new Windows Terminal app to be just perfect for handling multiple sessions at once.

![terminal snapshot](/notebooks/img/terminal.png)

In this article, we’ll explore some of the features and settings for the WT, and in this particular demonstration we will be using a Linux Virtual Machine as a guest to connect to, but we can set up automatic SSH sessions to routers, switches, FTP, etc.

So let’s get started.

For this example, I tested with a local virtual machine. Firstly, I chose a “Headless Start”, so it starts and runs in the background. This helps me keep things a little bit lighter and more organized. Let’s see:

![virtualbox](/notebooks/img/virtualbox.png)

Now, this is just a personal choice, and it may or may not work to you, depending on your needs. Personally, I don’t need a GUI for my server, so I can just ssh into it from any terminal.

Let’s now get the Windows Terminal app.
#  1. Download Windows Terminal from the Microsoft Store and run it.

It’s free and it’s lightweight so this can be setup in the coffee break and get back to work with a more organized desktop.

Just open the WT application and we’ll start from there.

Note: If you need to use Elevation mode, just type Win+R and write “wt”. Then ctrl+shift+Enter to open it with admin privileges.
# 2. Creating and customizing sessions through the settings file.

Now that I have a VM running in the background, I can then configure the Terminal so I can automate the connection (And of course, some appearance customization and additional settings) in no more than 5 minutes and open it with a couple of clicks from now on.

Firstly, open the settings.json file by clicking the down arrow and then “settings” (Or just hit Ctrl + ,) in the upper section:

![config file image](/notebooks/img/wt-config.png)

You can edit the settings.json file in Visual Code or any text editor.

Create a new field between “{ }” and enter the configuration you like (You’ll find the path to the official Microsoft WT documentation inside the settings.json file for a lot more information). I included a simple *commandline instruction* here to execute the ssh connection when this new tab opens. Now all the WT will do when I open this new tab is execute this command and Voilà, a new SSH session to my desired server is opened.

If **DHCP** is enabled, this setup won’t be as useful, and we would need to modify the settings.json file everytime the dhcp lease time expires.

Note also the colors I chose in the “foreground” and “tabColor” fields and even a CentOS icon in my Images directory. That’s entirely optional, but I set that up for convenience, since I will be configuring some different VMs from this, and coloring will be necessary when multiple sessions are open.

The order in which the new tab was added does matter. In my case, you can see I put the new one in the third place, just before the Azure Cloud Shell settings.

Double-check your syntax and Ctrl+G to save the file and move on.
# 3. Open the tab we just created

The results:

![Terminal image](/notebooks/img/wt-main.png)

And we’re done for this one session. Now we can add more SSH sessions and have them opened at the same time, all in a single application.

This was for an automatic VM-host communication session, but this can be done for a lot of purposes. There is also a very powerful WSL 1 & 2 integration (Which I am really enjoying), but that’s a topic for a future post.

There are a lot of options and features in this amazing new app by MS and this was just one, but I hope I succesfully conveyed the idea of what’s possible with it.

Cool, huh?

Original post @ Medium.com: https://medium.com/@armandogv25/keeping-multiple-ssh-sessions-neat-and-clean-30d493307238
