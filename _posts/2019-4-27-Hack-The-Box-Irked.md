---
layout: post
title: Hack The Box - Irked
---

My first [Hack The Box](https://www.hackthebox.eu/) system is in the books!

For my first Hack The Box (HTB) system, I chose "Irked".  Irked was labeled as a "easy to medium", so I thought it would be perfect to get my feet wet with HTB.  Overall, my impression of Irked was positive.  I thought the advertised difficulty was accurate: there were things I knew off the top of my head, and there were things I had to Google.  Without further delay, let's get root.txt!

## Where To Start
Tackling a HTB machine is always going to be a bit different than a production system at a company, but the methodology is roughly the same.  While there is some debate over the actual steps, they will typically contain the following phases: Recon, Scanning, Exploitation, and Post-Exploitation.  Some may add in phases like scoping or reporting, or have different names for these phases.  

## Scanning
Wait? What happened to the Recon phase?  This is one of those things that is different about a HTB challenge.  Since I already know what my target is, I don't really have to do recon, and can just move into scanning.  If there were an actual company target, or something I would do professionally, then I would be learning as much as possible about the company and their systems minus any actual network or physical interaction with the systems.

### Nmap
Nmap is typically the "go-to" first tool when conducting a pen test or a Capture the Flag (CTF).  On the HTB network, Irked has an IP address of 10.10.10.117. So let's get started…

![1](/images/irked/1.png)

**Image 1** - The Nmap command.

![2](/images/irked/2.png)

**Image 2** - The Nmap results.

When the Nmap completes, we see that we have a few ports open.  Port 80 (web) is one of those ports, so let’s take a look at that in our browser.

![3](/images/irked/3.png)

**Image 3** - The Irked homepage.

This being a practice machine, the “IRC is almost working!” is certainly a hint!  Looking back at our Nmap results from Image 2, we see that there are several [IRC](https://en.wikipedia.org/wiki/Internet_Relay_Chat) ports open.  Let’s work with 6697 which Nmap identified as “UnrealIRCd”.  A quick Google search for “unrealircd vulnerability” reveals an exploit in Metasploit!

![4](/images/irked/4.png)

**Image 4** - Google search results.

## Exploitation
Let’s fire up Metasploit and see if it is vulnerable.

![5](/images/irked/5.png)

**Image 5** - The Metasploit banner upon launching (one of them anyway…).

We learned from our Google search that the exploit we want is located at “exploit/unix/irc/unreal_ircd_3281_backdoor”.  Run the “use” command with that path to set the exploit.  Set the “RHOST” as the IP of Irked, the “RPORT” as 6697, and the “LHOST” as your IP address.

![6](/images/irked/6.png)

**Image 6** - The Metasploit parameters.

Next, we will just issue the “run” command and see what we get back.

![7](/images/irked/7.png)

**Image 7** - Running our exploit.

We have a shell!!!  We can run the “whoami” command to see what account we are running commands as, in this case the account is “ircd”.  Looking around the system a little bit we can see that there is another user named “djmardov”.

![8](/images/irked/8.png)

**Image 8** - Looking for users in the “/home” directory.

Let’s explore what is in djmardov’s directory.  Inside we see two files: the user.txt file (which is one of the files we need to read to earn HTB points) and a hidden “.backup” file.

![9](/images/irked/9.png)

**Image 9** - The two files and their permissions.

Looking at the permissions for the two files, the only one we can do anything with right now is the “.backup” file.  Let’s take a look at the file.

![10](/images/irked/10.png)

**Image 10** - The contents of the “.backup” file.

Well, that is convenient!  It looks like this password is the [Konami Code](https://en.wikipedia.org/wiki/Konami_Code) written out.  It says it is a “steg” password.  I’m sure that they mean [steganograhpy](https://en.wikipedia.org/wiki/Steganography)!  But for this to be related to steg I would need a picture right?  Wait… Wasn’t there a big obnoxious angry face image on the home page?  Yup!  Let’s download that and take a look at it.

![11](/images/irked/11.png)

**Image 11** - The “irked” image.

Author’s note: There are a number of linux tools that can do steg.  I tried to use [StegoSuite](https://stegosuite.org/) on the command line but it refused to take the password.  Luckily, there is an [online tool](https://futureboy.us/stegano/decinput.html) that can do it as well!  Let’s navigate to that page, upload our image, and put in the password.  Do NOT do this with company or customer data, this is just for fun.

![12](/images/irked/12.png)

**Image 12** - All of the information needed to extract any data hidden in the image.

When you click the “Submit Query” button, let’s see if we get anything back…

![13](/images/irked/13.png)

**Image 13** - The extracted data.

Awesome!  It appears to have worked!  My guess is that this is the password to the “djmardov” account.  Let’s try to SSH in as “djmardov”.

![14](/images/irked/14.png)

**Image 14** - Logging in as “djmardov”.

We are in!!!  Let’s make sure that we are in the right directory and then read that “user.txt” file so we can plug it into HTB.

![15](/images/irked/15.png)

**Image 15** - The contents of the “user.txt” file.

## Post-Exploitation

Now that we have at least locked down a low privileged user account, we are ready to move on to root!
I will save you some time navigating around, the interesting thing to look for is a binary called “viewuser”, which is located in /usr/bin/.  This can be found by looking for items that have the [SUID](https://en.wikipedia.org/wiki/Setuid) set.
![16](/images/irked/16.png)

**Image 16** - A listing of files with SUID.

The “viewuser” binary can be executed by anyone as noted by it’s permissions.

![17](/images/irked/17.png)

**Image 17** - The permissions of the “viewuser” binary.

If we actually read the “viewuser” binary, it is full of non-printable characters.

![18](/images/irked/18.png)

**Image 18** - The contents of the “viewuser” binary.

Let’s see if we can make heads or tails of it with the “strings” command. Strings will just print the readable characters.  It looks like the “viewuser” binary makes a call to “/tmp/listusers”.

![19](/images/irked/19.png)

**Image 19** - A more readable “viewuser” binary.

Looking at the permissions on the “/tmp” directory, it appears that it is world-writable!

![20](/images/irked/20.png)

**Image 20** - “/tmp” is world-writable.

For a quick recap: there is a binary (that runs as root) that we can execute as a low privileged user, that binary calls a file that we can edit.  At this point we have a choice: 1) We can use this file to just read the contents of the root.txt file if that is all you care about for HTB points purposes, or 2) we can have it create a reverse shell back to our attacking box.  For either to work, we need to edit the “listusers” file.  We will use “nano” just to upset the “vi” geeks. :)

![21](/images/irked/21.png)

**Image 21** - Using “nano” to edit the file for our own evil purposes.

Let’s do option 1 first, where we just edit the file so that it shows us the contents of root.txt.

![22](/images/irked/22.png)

**Image 22** - Creating a simple bash script to read the contents of root.txt.

In order for this to work, we will have to make “listusers” executable.

![23](/images/irked/23.png)

**Image 23** - Making “listusers” executable.

When we run the “viewuser” binary, the contents of the root.txt file are appended to the results!

![24](/images/irked/24.png)

**Image 24** - The contents of root.txt is included along with the regular output of “viewuser”.

If you want to go the reverse shell route, that is also doable with the listusers file!  First, setup your listener on the system you are attacking from.

![25](/images/irked/25.png)

**Image 25** - The listener setup on our attacking box.

Edit the “listusers” file the same way as you did in Image 21.  This time, use your favorite python reverse shell.  There are plenty of publicly available python reverse shells, the one I am using is from [PenTestMonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).  Make sure you put in the IP address of the system you are attacking from.

![26](/images/irked/26.png)

**Image 26** - The python reverse shell.

Again, you will have to make “listusers” executable.

![27](/images/irked/23.png)

**Image 27** - Making “listusers” executable.

Run the “viewuser” command again, as you did in Image 24.  You should now see that a reverse shell was opened as root.  You will now be able to execute commands as root, including reading root.txt

![28](/images/irked/28.png)

**Image 28** - The contents of root.txt.

## Reporting
All good penetration tests have one thing in common: good reports.  This is a *very* simplified version of how I would write the Irked findings if I were submitting them.

### Finding 1 - Vulnerable IRC Package
__Severity__: CVSS:2.0/AV:N/AC:L/Au:N/C:P/I:P/A:P - High (I am using CVSSv2 here as that is how the [CVE for this issue](https://www.cvedetails.com/cve/cve-2010-2075) was originally scored.)

__Summary__: It was found that the “Irked” system was employing a vulnerable version of Unreal IRC.  Vulnerable versions of software can have publicly available exploit code that attackers can use to gain unauthorized access to systems.  It is recommended that Irked system administrators update to a non-vulnerable version of Unreal IRC.

### Finding 2 - Steganography
__Severity__: Medium

__Summary__: It was found that the “Irked” homepage contained an image with embedded steganographic data.  Steganography is the practice of concealing data within another file.  It is recommended that the embedded data be removed and that Irked system administrators discontinue the practice of embedding data in images.

### Finding 3 - Improperly Configured Permissions
__Severity__: Medium

__Summary__: It was found that the “viewuser” binary, which runs as root, also calls a world-writable file: “listusers”.  Attackers could escalate their privilege by editing the “listusers” file to run malicious code.  Recommendation is to properly limit the access to the “listusers” file.

### Attack Chain
Nmap yielded port 80 > Homepage expressed frustration over IRC, which Nmap confirmed with port 6697 > Exploit in UnrealIRC led to a reverse shell > Found steganography password > Extracted data from “irked.jpg” > Logged in as “djmardov” > Found binary which called a world-writable file > Elevated privileges to root.

## Conclusion
I hope you enjoyed the write up and the system!  Feel free to reach out to me on LinkedIn with any questions!

