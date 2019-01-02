---
layout: post
title: VulnHub - Bulldog 1
---

A fun [VulnHub](https://www.vulnhub.com/) system, and now my first blog post are in the books!

For this VulnHub system, I chose "[Bulldog 1](https://www.vulnhub.com/entry/bulldog-1,211/)".  Bulldog 1 was created by my friend and colleage [Nick Frichette](https://frichetten.com/). Bulldog 1 was labeled as a "Beginner / Intermediate", making it perfect for someone who is new to penetration testing.  Overall, my impression of Bulldog 1 was positive.  I thought the advertised difficult was accurate: there were things I knew off the top of my head, and there were things I had to Google.  Without further delay, let's get root!

## Where To Start
Tackling a VulnHub machine is always going to be a bit different than a production system at a company, but the methodology is roughly the same.  While there is some debate over the actual steps, they will typically contain the following phases: Recon, Scanning, Exploitation, and Post-Exploitation.  Some may add in phases like scoping or reporting, or have different names for these phases.  

## Scanning
Wait? What happened to the Recon phase?  This is one of those things that is different about a VulnHub challenge.  Since I already know what my target is, I don't really have to do recon, and can just move into scanning.  If there were an actual company target, or something I would do professionally, then I would be learning as much as possible about the company and their systems minus any actual network or phyiscal interaction with the systems.

### Nmap
Nmap is typically the "go-to" first tool when conducting a pen test or a Capture the Flag (CTF).  On my local network, Bulldog 1 recieved an IP address of 192.168.56.4.  So lets get started...

```
nmap -vvv -sT -sV 192.168.56.4  --open -oA bulldog_nmap
```
