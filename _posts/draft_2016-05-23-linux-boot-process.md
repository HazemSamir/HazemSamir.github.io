---
layout: post
title: "Linux boot process in a nutshell"
date: 2016-05-22
type: tech
excerpt: "what happens in the background from pressing the power button till the prompt"
tags: [espace, intern, exam, linux, boot, grup, bios]
comments: true
---

In my last post about [espace exam](http://hazemsamir.github.io/espace-intern-exam/), I mentioned a question about linux boot process. It was one of questions that I could not answer so I watched many videos and read many articles explaining the process. and this what I understand so far:

> ## Explain how the linux boot works as much detailed as possible starting from pressing the power button.


# The very short answer:

We can summarize the process in 5 steps [^wiki]:

1. The BIOS initializes the necessary hardware for boot then starts the boot loader from the configured boot device.
2. The boot loader loads the kernel into memory and gives it control.
3. The kernel setups the system (memory and driver initialization) then starts the init process.
4. The init consists of levels of shell scripts, each of which consists of a set of run-level programs.
5. Forming the user environment. For desktops (starts the GUI and login and session manager etc).

<figure>
	<img src="https://qph.is.quoracdn.net/main-qimg-8ae8f8f82201d9a309b1bbe608a082f9" style ="width:400px">
</figure>


# The long Answer:

Let's discuss briefly each step and its terms and what they mean.
{::comment}
Well, till now I am just copying and pasting from articles on the web. so let's stop copying and try to explain things in a new way.
{:/comment}

## 1. BOIS:


## FootNotes
[^wiki]: from [wikipeadia](https://en.wikipedia.org/wiki/Linux_startup_process#Overview)
