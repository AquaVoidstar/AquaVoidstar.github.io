---
layout: post
title: Configuring Ubuntu
tags: [Ubuntu LTS, Troubleshooting]
author: MStoecker
date: 2020-06-04
image: "assets/img/thumbnails/UbuntuFight.png"
thumbnail: "assets/img/thumbnails/UbuntuFight.png"
feature-img: "assets/img/thumbnails/UbuntuFight.png"
bootstrap: true 
excerpt_separator: <!--more-->
---

* TOC
{: toc}

Partly to preserve my own sanity after figuring out all of this for the second time
I wanted to list exactly what I did to get Ubuntu 20.04 LTS to work. This is by no means
an indictment of the OS- I love using it and its my primary OS.<!--more-->
I also feel like most of these problems are pretty unique and I desperately hope that no one suffers this much. All this being said, I am in no way an expert at these subjects and this is just what I found that worked when I was scavenging forum posts.
Firstly my computer build:

Object | Type
----------|---------------------------------------------------------------------
CPU: | Intel Core i5-9600K 3.7 GHz 6-Core Processor
CPU Cooler: |Corsair H100i PRO 75 CFM Liquid CPU Cooler
Motherboard: |Gigabyte Z390 UD ATX LGA1151 Motherboard
Memory:| GeIL EVO SPEAR 16 GB (2 x 8 GB) DDR4-3000 CL16 Memory
Storage:| Silicon Power A55 512 GB 2.5" Solid State Drive
GPU: |Gigabyte GeForce RTX 2070 8 GB WINDFORCE 2X Video Card
Power Supply:| Corsair TXM Gold 550 W 80+ Gold Certified Semi-modular ATX Power Supply



# Problem 1: Ubuntu won't install correctly
I had this problem also with Ubuntu LTS 18, when you get to the basic installation page, if you try to install you will eventually have a permanent loading mouse icon. Surprise! I already have failed. This honestly must be a world record. Going to the tty2 (ALT+CTR+F2), you will see a repeated output that will not allow for any input.

<img src="https://i.stack.imgur.com/rGO0I.jpg" alt="enter image description here">

This link [HERE](https://askubuntu.com/questions/771899/pcie-bus-error-severity-corrected)
provides the solution that worked for me but the details are basically:

1. Shutdown you coputer.
2. While power on, hold shift to enter recovery mode
3. Here press e to edit the boot up process.
4. Go to the line that starts with linux and enter pci=nomsi OR pci=noaer to the end of the line
    1. I used pci=nomsi
5. You should be able to install Ubuntu now. This does not make the change permanent so when it asks you to reboot, you have to preform this process again.

To make this permanent, grab the grub file:

 ```console
 foo@bar:~$ sudo nano /etc/default/grub
  ```

Go to the line "GRUB_CMDLINE_LINUXGRUB_CMDLINE_LINUX_DEFAULT_DEFAULT" and then append pci=nomsi to the end. Finally update the grub setting with:

 ```console
 foo@bar:~$ sudo update-grub
  ```

  This should solve the most immediate problem and allow for a working OS.

# Problem 2: Unable to pass screenlock
After attempting to enter the corect pasword to login to my primary user, I decided first to reinstall Ubuntu to see if that would fix the problem. It did not. 
I Did still have access to the tty2 (CTR+ALT+F2) so I tried changing my password. This also did not work. Going to the forums: ([HERE](https://askubuntu.com/questions/509834/lock-screen-does-not-unlock-with-correct-password-gnome-and-ubuntu-14-04
)) suggested I attempt to fix this by modifying the ownership of the unix_chkpwd with:

 ```console
 foo@bar:~$ sudo chown root:shadow /sbin/unix_chkpwd
 foo@bar:~$sudo chmod 2755 /sbin/unix_chkpwd
  ```
or run these lines:

 ```console
foo@bar:~$ sudo chown root:shadow /etc/gshadow
foo@bar:~$ sudo chown root:shadow /etc/gshadow-
foo@bar:~$ sudo chown root:shadow /etc/shadow
foo@bar:~$ sudo chown root:shadow /etc/shadow-
  ```
but also no dice. Finally, I did the one unthinkable thing. I made another user. This of course instantly fixed the problem by avoiding it so it's a win... I guess. I still do not know how to fix this lock-screen error which is pretty disapointing.