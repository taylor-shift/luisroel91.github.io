---
layout: post
title: 'PMAT: SillyPutty'
date: 2023-06-22 20:22 -0700
categories: [writeups]
tags: [malware analysis, security, writeup]
---
I've been taking [TCM's](https://academy.tcm-sec.com/) practical malware analysis and triage course, extremely fun and interesting so far.

This is a little writeup I'm doing for the first challenge called SillyPutty.

Just a heads up...this is live malware we'll be dealing with. If you'd like to follow along with this post, you can access the lab materials [here](https://github.com/HuskyHacks/PMAT-labs/).

> Please make ABSOLUTELY SURE you are not running malware on your own machine, outside of a proper environment! Setup [FlareVM](https://github.com/mandiant/flare-vm) and [REMnux](https://remnux.org/)(each on their own VM) and only connect them to each other in an isolated network. This post is for informational purposes only. You are responsible for your own actions.
{: .prompt-danger}

After you extract the malware, you'll see the executable is called `putty.exe`...it is masquerading as putty, a popular ssh client.

The excercise has the following questions, let's answer them:

## Basic Static Analysis

1. What is the SHA256 hash of the sample

Hashes serve as an easy way to 'fingerprint' malware without executing it.

There are several tools to get this info...but the simplest one is sha256sum.exe:

![getting sha256 hash]({{site.url}}/assets/img/pmat-challenge-1.md_files/Screenshot from 2023-06-22 18-08-40.png)

A more comprehensive one would be [PEStudio](https://www.winitor.com/), which I'll use to answer the question below.

2. What architecture is this binary?

![more info with PEStudio]({{site.url}}/assets/img/pmat-challenge-1.md_files/Screenshot from 2023-06-22 18-16-46.png)

The first bytes (MZx) tell us this executable is in PE format (as opposed to something like ELF on Linux or Mach-O on Darwin), its also a 32-bit binary. 

3. Are there any results from submitting the SHA256 hash to VirusTotal?

VirusTotal is a website where people submit hashes of malware samples in the wild. It helps researchers and AV companies quickly share signatures, which helps AV develop rules against novel malware more quickly.

* If you submit a sample, it will run it against AV engines and see what was detected. 
* If you lookup a hash (or other identifier), it will tell you if it encountered any samples matching that identifier, speeding up the process of identifying malware.

Lets submit our hash to VT and see what we get:

![VT Results]({{site.url}}/assets/img/pmat-challenge-1.md_files/Screenshot from 2023-06-22 18-27-07.png)

We have many hits. Keep in mind, this might not always be the case. This particular sample is well known since many people have followed this lab and submitted the same sample.

4. Describe the results of pulling the strings from this binary. Record and describe any strings that are potentially interesting. Can any interesting information be extracted from the strings?

Strings contained within a binary can often help us gleam some info about its mechanism of attack. On the simpler end, there are `FLOSS.exe` and the built in `strings` command...but the ouput of that can be extremely messy. For this task, I'd rather use PEStudio.

Most of it just seems like what you would expect...strings that the program would use normally. I can't really find any identifiers...all the URLs seem to be official ones.

5. Describe the results of inspecting the `IAT` for this binary. Are there any imports worth noting?

I'll continue to use `PEStudio` for this question. The `IAT` stands for `Import Address Table`. Its how the binary knows the addresses for functions it might call, functions which are located within `DLLs` it might depend on.

I saw some stuff relating to handling registry keys...but even if it wasn't malware, Putty does this normally so I wouldn't really call it an identifier.

6. Is it likely that this binary is packed?

When a binary is packed, it means that it has compressed hidden data inside of it, which it will extract and load into memory at runtime. Its an evasive technique, but is very easy to detect during manual analysis.

How can we tell?

The binary is divided into different sections (.text, .rsrc for example)

![binary sections]({{site.url}}/assets/img/pmat-challenge-1.md_files/Screenshot from 2023-06-22 18-57-14.png)

Notice the virtual-size and raw-size properties...if you notice a large difference between these two values for any table (but in particular for the .text section, it contains the code in a binary), it would be worth further investigation because the binary might be packed.

## Basic Dynamic Analysis

Ok, here is where things get a bit more interesting (imo).

In this section, we'll detonate the malware sample and use various tools to trace its behavior.

> Before we start, make sure you save a snapshot of FlareVM so you can easily revert once you've detonated the sample. This allows us to have a clean slate for every detonation so that we don't contaminate our analysis.
{: .prompt-warning}

7. Describe initial detonation. Are there any notable occurances at first detonation? Without internet simulation? With internet simulation?

Notice this question asks about internet simulation. This is why its good to have a proper environment setup for analysis, as it will make our lives easier. :)

Run `inetsim` on your REMNux machine:

![inetsim screen]({{site.url}}/assets/img/pmat-challenge-1.md_files/Screenshot from 2023-06-22 19-11-03.png)

Now go to your FlareVM machine and set its DNS to the IP of your REMNux machine. You can do so in control panel -> network and sharing center -> change adapter settings:

![Windows network settings]({{site.url}}/assets/img/pmat-challenge-1.md_files/Screenshot from 2023-06-22 19-16-51.png)

Once you've done that, also run `wireshark` on your REMNux machine and start capturing traffic.

Ok...we're setup, lets detonate the sample and see what happens. On our FlareVM, all we saw was a blue screen that popped up for a split second and then a regular putty window...that blue screen looked to me like a Poweshell terminal.

We also got some interesting traffic on wireshark:

![wireshark capture]({{site.url}}/assets/img/pmat-challenge-1.md_files/Screenshot from 2023-06-22 19-21-13.png)

8. What is the DNS record that is queried at detonation?

It seems the malware caused a DNS request to the domain `bonus2.corporatebonusapplication.local`...

9. What is the callback port number at detonation?

Port 8443

10. What is the callback protocol at detonation?

SSL/TLS

Lets revert our VM to an earlier snapshot and try some deeper analysis on the host to see if we can figure out what exactly is calling out to this domain.

For this we'll use Procmon from the [Sysinternals](https://learn.microsoft.com/en-us/sysinternals/) suite. Open Procmon and detonate the malware.

If we filter by process name, we get a lot of results:

![procmon]({{site.url}}/assets/img/pmat-challenge-1.md_files/Screenshot from 2023-06-22 19-41-57.png)

Most of this isn't anything out of the ordinary...except it opened and closed a lot of different threads, all under the same PID...This could mean that it opened sub-processes.

Let's check the process tree to see what opened up at the same time:

![process tree]({{site.url}}/assets/img/pmat-challenge-1.md_files/Screenshot from 2023-06-22 19-44-54.png)

There it is! We can see our binary also started up powershell, which in turn started a host process...more interestingly is that we can see the command being ran:

![shellcode]({{site.url}}/assets/img/pmat-challenge-1.md_files/Screenshot from 2023-06-22 19-47-00.png)

Seems like its something that's base64 encoded, which is then decoded and ran. AKA shellcode. Its trying to load its second stage from the domain mentioned above, which means we should be able to intercept the call using our REMNux VM.

11. Attempt to get the binary to initiate a shell on the localhost. Does a shell spawn? What is needed for a shell to spawn?

It does not...I am assuming that it fails because it can't complete the SSL handshake. Which is why it freaks out and retransmits the same packet a few times in the screenshot above.

Lets run an `ncat` listener in SSL mode using REMNux to 'catch' the shell:

`ncat -nvlp 8443 --ssl`

Then run the malware again, if our assumptions are correct, we'll catch a shell:

![caught a shell]({{site.url}}/assets/img/pmat-challenge-1.md_files/Screenshot from 2023-06-22 20-13-02.png)

I hope this basic intro to malware analysis has been fun to follow along with!

See you soon,
- L















