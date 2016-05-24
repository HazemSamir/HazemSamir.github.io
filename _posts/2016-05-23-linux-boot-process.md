---
layout: post
title: "Linux boot process in a nutshell"
date: 2016-05-24
type: tech
excerpt: "what happens in the background from pressing the power button till the prompt"
tags: [espace, intern, exam, linux, boot, grup, bios]
comments: true
---

In my last post about [espace exam](http://hazemsamir.github.io/espace-intern-exam/), I mentioned a question about linux boot process. It was one of the questions that I couldn't answer then, so I watched many videos and read articles explaining the process.

Well, basically the rest of the post is kind of reformatting what I understood from wikipedia articles and some youtube videos, so don't bother seeing some text copied and pasted from some where =).

> ## Explain how the linux boot works as much detailed as possible starting from pressing the power button.


The short answer:
================

We can summarize the process in the following steps:

1. The BIOS initializes the necessary hardware for boot then starts the boot loader from the configured boot device.
2. The boot loader loads the kernel into memory and gives it control.
3. The kernel setups the system (memory and driver initialization) then starts the init process.
4. The init consists of levels of shell scripts, each of which consists of a set of run-level programs.
5. Forming the user environment. For desktops (starts the GUI and login and session manager etc).

<figure>
	<img src="https://qph.is.quoracdn.net/main-qimg-8ae8f8f82201d9a309b1bbe608a082f9" style ="width:400px">
</figure>



The "Quite" Long Answer:
========================

## 1. BIOS:

**BIOS** stands for _**B**asic **I**nput **O**utput **S**ystem_. Obviously it is the first thing to work after pressing the power button. It is OS independent (depends on the hardware). [Read: Where is BIOS replaced?](#where-is-bios-replaced)

Everyone has an experience with BIOS ugly black screen that shows up when you turn on the computer and the option to press delete, alt or f-keys to get you into a scary text screen with complicated settings.

**Briefly BIOS has two basic tasks:**

1. Runs a POST (Power On Self Test) checking different input/output devices and the connected drivers, making sure every thing is well for the next step.
2. Searches sequentially for a bootable driver (that contains a valid boot loader program) then execute the boot loader from it.

[Read: Change the BIOS search order](#change-the-bios-search-order)

[Read: How the BIOS check a valid bootable driver?](#how-the-bios-check-a-valid-bootable-driver)

## 2. Boot Loader:

The first sector of the bootable disk is called MBR (Master Boot Record) as it stores information about the boot loader.
There are many types of boot loaders and some of them may have one or more stages. I am goint to talk about **GRUP** as it the one used on my machine:

**GRUP** stands for _**GR**and **U**nified **B**ootloader_ ([see what are the advantages of GRUB?](#what-are-the-advantages-of-grub)). It has two major versions _GRUB 1_ and _GRUP 2_. The later adds the features of detecting different OS installed on the machine automatically and gives the user the option to decide which OS to boot up.

After that, the Linux kernel is loaded into the memory and is given control.

## 3. Kernel:

An OS Kernel is a part of the system responsible for controlling/interacting with the hardware, memory management and many other low level stuff.

[Read: What Linux really is?](#what-linux-really-is)

Linux kernel is loaded into memory ([see appendix](#kernel-loading-into-memory)) then calls the setup() function that initializes memory and configures the various hardware (processors, I/O, and storage devices).

Then it calls the Init process as first process to be run in the system.

## 4. Init

**Init** is short for _initialization_. Typically, the Init process can be find in the directory `/sbin/Init`.

[Read: What a process is?](#what-a-process-is)

Init's job is to get everything running the way it should be. Essentially it establishes and operates the entire user space. This includes checking and mounting file systems, starting up necessary user services, and ultimately switching to a user-environment when system startup is completed.


[Read: Are there variations of init?](#variation-of-init)


## Finaly

By this time the user would be prompted to enter the credentials, after launching the different services for the GUI.

--------------------------------------------

You can get a quick _visual_ overview by watching this [9 mins youtube video](https://www.youtube.com/watch?v=y3IbvEjXF5M).


__If you reached here, my congratulations for you. You are a Hero! =D if you still want to get more information read the appendix.__


--------------------------------------------


Appendix
========

## Where is BIOS replaced?

Historically it is was placed on a ROM(read only memory) and to change it you have to change the memory chip. Then in modern computers, it is stored in a flash memory allowing it to be changed, configured, updated many times without replacing any chips.

## Change the BIOS search order

You may be familiar with that if you tried to install an OS on a machine before, first you connect a flash memory or a CD disk that contains an image of your OS. Then you can change the order the BIOS searching the drivers so that it boots from your removable disk first instead of the local hard disk. This can be done from the interactive BIOS settings window.

## How the BIOS check a valid bootable driver?

When the BIOS searches a driver, basically hard disk or (flash memory), it looks at the first sector of it. It checks the first bytes that works as boot loader flags. If those bytes were valid, it considers the driver bootable and switch to the boot loader, else it considers the drive as unbootable.

## What are the advantages of GRUB?

It's a free software from the GNU project. One of its advantages that it reads its configuration from file-system rather than being embedded with the MBR. This allows the configuration file to represented in a human-readable format and to be modified easily by the user in the run time (find the conf file in `/boot/grub/grub.conf`). The GRUB also provides a basic command line utility that can be used to fix things if there where kind of errors or mis-configurations.

___As a quick note:___ there is another type of boot loaders like SYSLINUX/ISOLINUX which used for booting from the FAT file-systems (and used to boot from live USBs if you tried it before).

## What Linux really is?

Well Linux is actually the name of the kernel (not the whole OS). The reset of the programs come from different resources, mainly the GNU project, so some of people insist on using the term GNU/Linux naming the whole OS not just Linux. You can read more about [GNU/Linux naming controversy](https://en.wikipedia.org/wiki/GNU/Linux_naming_controversy).

## Kernel loading into memory

A full version of Linux kernel is around 70 MB in size. It may be loaded into memory by either reading it directly form the file-system (slower) or read a compressed image of it then use the CPU to decompress the image into the memory (faster, less file-system access and usage).

## What a process is?

Briefly, in Linux when you run a program, it launches a group of one or more processes (you can see them by the command `ps -ef`). Any process has exactly one parent process (the one that called it or init) and one or more child processes (processes called by it). So it is some thing like a tree of processes, each process has an unique ID number called PID.

Init is the root process (parent of all other processes in the system) with no parent process and its PID=**1**, and eventually the last process to terminate (while shutting down).

## What is run levels of Init?

Init process has many run levels of shell scripts to be run, the run level is passed to it as a parameter from the kernel. Typical init has 7 run levels and we care about:

* 0 : halt
* 3 : full multi user
* 5 : X11 (GUI server)
* 6 : reboot

## What is kernel panic?

It happens when the kernel can not call the init process due to misconfiguration or missing the process file.

## Variation of init:

Traditionally, one of the major drawbacks of init is that it starts tasks serially, waiting for each to finish loading before moving on to the next. When startup processes end up I/O blocked, this can result in long delays during boot. So variation of the init process is appeared to address this and other design problems like (systemd, upstart, initng etc).
