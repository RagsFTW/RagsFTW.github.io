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
nmap -vvv -p- -sT -sV 192.168.56.4  --open -oA bulldog_nmap
```
When the Nmap completes, we see that we have two web ports open at port 80 and 8080.  I will save you some time, these are the same.  There are no differences between the sites hosted on the two ports.

(Image 1)
**Image 1** - The Nmap results.

Lets move on to some quick web "recon"...

### Dirb
Generally when i see something open on port 80, I will fire up a browser and see what I am dealing with...

(Image 2)
**Image 2** - The Bulldog 1 homepage.

In the real world I may not use a directory scanner, they are noisy and in rare instances could cause impacts.  On a VulnHub system, it is OK to "go loud".  Dirb is pretty simple, just give it a target and the path to a world list and you are good to go!

```
dirb http://192.168.56.4 /usr/share/wordlists/dirb/small.txt -r
```
When dirb finihses, we can see that it found a "/admin" directory and a "/dev" directory, so let's check it out!

(Image 3)
**Image 3** - The dirb results.

### Digging Into The Webpage
Now that we have a listing of pages, let's take a look at each.

(Image 4)
**Image 4** - The /admin page.

Here on the /admin page we see a login prompt for Django.  Note this page for later...

(Image 5)
**Image 5** - The /dev page.

Here on the /dev page, at the bottom, we see some key pieces of information: email addresses and a link to a Web Shell.

(Image 6)
**Image 6** - The Web Shell page.

Quickly clicking on the Web Shell link we are greated by a page that says we need to authenticate.  That's OK, let's go back to the /dev page...

At the bottom of the page we saw email addresses, or should I say usernames!  This is perfect for making a list of potential usernames to use in our attacks.

At this point we have some options: 1) brute force the login page, 2) crack the users passwords.  Let's go with option 2.  "But i don't have and passwords!" you may say.  But if we take a look at the source of the /dev page we will see some hashes!!!

(Image 7)
**Image 7** - The source of the /dev page.

You may think that this is crazy, but this is something I have seen in the wild!  You *could* brute force the login via a public wordlist or a specially crafted wordlist but it would take some time.  If you do, want to go the crafted wordlist route, your base words are pretty obvious.  :)

## Exploitation
Now we have moved out of the realm of scanning and into exploitation!  First thing that we need to do is crack these hahses...

### Hashcat
In order to crack these hashes, we first have to do a few things.  The first of which would be to identify the kind of hash we are going to crack.  Using a tool called "hashid", we can identify the hash so we can tell hashcat how to crack the hash.  Copy one of the hashses to your clipboard and then paste it into a terminal after the hashid command.

```
hashid 6515229daf8dbdc8b89fed2e60f107433da5f2cb
```

(Image 8)
**image 8** - The results of the hashid command.

I am going to save you some time and tell you that is is in fact SHA-1.  The next thing that we should do to make our lives easier is to create a text file with all of the discovered hashes in it.

(Image 9)
**image 9** - The text file with all the hashes named hashes.txt

Now we have our hashes and we know what kind of hashes they are, we are ready to start cracking!  We have to build our command for hashcat...

```
hashcat -a 0 -m 100 hashes.txt /usr/share/wordlists/rockyou.txt --force --potfile-disable
```
Let's break down that command: "hashcat" initiates hashcat, "-a 0" specified the attack mode (0 tells hashcat to do a straight hash to hash comparison, and "-m 100" specifies the hash type (100 tells hashcat to do SHA-1 hashes).  The next section specifies the path to your hash file, and then the path to the wordlist comes next.  After this, I usually tack on any other options.  In this case I am using "--force" due to some errors on my Kali VM and "--potfile-disable" so that I will be able to see the cracked passwords in the output and not store them in my potfile.

(Image 10)
**image 10** - The cracked hashes.

More to come!!!
