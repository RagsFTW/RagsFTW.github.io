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

Lets move on to some quick web scanning...

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
**Image 8** - The results of the hashid command.

I am going to save you some time and tell you that is is in fact SHA-1.  The next thing that we should do to make our lives easier is to create a text file with all of the discovered hashes in it.

(Image 9)
**Image 9** - The text file with all the hashes named hashes.txt

Now we have our hashes and we know what kind of hashes they are, we are ready to start cracking!  We have to build our command for hashcat...

```
hashcat -a 0 -m 100 hashes.txt /usr/share/wordlists/rockyou.txt --force --potfile-disable
```
Let's break down that command: "hashcat" initiates hashcat, "-a 0" specified the attack mode (0 tells hashcat to do a straight hash to hash comparison, and "-m 100" specifies the hash type (100 tells hashcat to do SHA-1 hashes).  The next section specifies the path to your hash file, and then the path to the wordlist comes next.  After this, I usually tack on any other options.  In this case I am using "--force" due to some errors on my Kali VM and "--potfile-disable" so that I will be able to see the cracked passwords in the output and not store them in my potfile.

(Image 10)
**Image 10** - The cracked hashes.  So, my Kali VM had an issue with Hashcat, I kept getting segfaults and it [drove me crazy](https://xkcd.com/290/) to the point where I just went to [Crackstation](https://crackstation.net/) to get what I needed for this blog post.  The syntax above is correct for Hashcat, hopefully your's works.  I would **NOT** reccomend using Crackstation in a corporate environment.

### Logging In and Looking Around
Now that we have the passwords to the "Nick" and "Sarah" accounts, let's find out what they go to!  My money is on the Django login page.  Let's try Sarah's credentials first and see what we get...

(Image 11)
**Image 11** - Sarah's credentials in the login page.

(Image 12)
**Image 1** - We're in!

What was that I remember about the Web Shell saying I need to be authenticated?  Let's check back there...

(Image 13)
**Image 13** - The now accessible web shell.

Boom! We have a nice foothold in our access to the Web Shell.  I will save you some time and tell you that the commands listed are the only commands you can run... at first.  Let's run a few quick commands... 

(Image 14)
**Image 14** - Running pwd.

(Image 15)
**Image 15** - Running ls -lart.

One way that you can run multiple commands in linux is to use "&&".  This will run your first command, and then the second.  "Whoami" is not a permitted command in the list, but let's see if we can get it to run by appending it to a valid command...

(Image 16)
**Image 16** - Successfully appending the "whoami" command.

It works!  We can now run essentially any command we want on the system as the "django" user.  Remember back in Image 15 we saw a "manage.py" script?  That tell me that python is installed on the system.  Also, "echo" is a permitted command.  You see where I am going with this?  Let's use "echo" to create a python script that will create a reverse shell back to us, and take advantage of the "&&" issue to change the permissions on that file, and then run the script...

### Reverse Shell
There are plenty of publicly available python reverse shells, the one I am using is from [PenTestMonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).  My reccomendation is to create a text file with the "echo" command ready to go for each line of the reverse shell.  This makes it simple to copy over to the system through the Web Shell.  Make sure you replace the IP address with your own IP.

(Image 17)
**Image 17** - The reverse shell, ready to go.

After we echo each line to the "shell.py" file we created, let's "cat" the file to make sure it looks right.

(Image 18)
**Image 18** - The reverse shell looks good to me!

We need to change the permissions on the "shell.py" file to make it executable, so let's append a command to do that...

(Image 19)
**Image 19** - Changing the permissions with the "chmod" command.

Let's setup our listener on the system we are attacking from...

```
nc -nlvp 1234
```

Now we can go back to the Web Shell and have the system run our malicious python script...

(Image 20)
**Image 20** - Running the malicious python script.

Let's check on our listener... and we have a reverse shell!  let's run a few quick commands to see who and where we are...

(Image 21)
**Image 19** - The reverse shell.

## Post-Exploitation
Now that we are inside the system, we need to look for ways to escalate our privilege and become root!



More to come!!!
