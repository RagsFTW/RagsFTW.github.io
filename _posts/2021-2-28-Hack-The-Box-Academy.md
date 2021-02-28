---
layout: post
title: Hack The Box - Academy
---

Another [Hack The Box](https://www.hackthebox.eu/) system is in the books!

![Title](/images/academy/Title.PNG)

For this Hack The Box (HTB) system, we chose “Academy".  Who is this “we” you speak of?  This HTB system was used as a training and mentoring aid for two of our co-workers who wanted to increase their offensive knowledge.  Without further delay, let's get root.txt!

## Where To Start
Tackling a HTB machine is always going to be a bit different than a production system at a company or customer location, but the methodology is roughly the same.  While there is some debate over the actual steps, they will typically contain the following phases: Recon, Scanning, Exploitation, and Post-Exploitation.  Some may add in phases like scoping or reporting, or have different names for these phases.  

## Scanning
Wait? What happened to the Recon phase?  This is one of those things that is different about a HTB challenge.  Since I already know what my target is, I don't really have to do recon, and can just move into scanning.  If there were an actual company target, or something I would do professionally, then I would be learning as much as possible about the company and their systems minus any actual network or physical interaction with the systems.

### Nmap
Nmap is typically the "go-to" first tool when conducting a pen test or a Capture the Flag (CTF).  Our target has an IP address of 10.10.10.215.  So let's get started…

![1](/images/academy/1.png)

**Image 1** - The nmap command.

![2](/images/academy/2.png)

**Image 2** - The nmap results.

The only ports that are open are port 22 (SSH) and port 80 (web).  That is a pretty small attack surface, so let’s focus on the web page first.  That is probably where the author of this system wants us to look…

![3](/images/academy/3.png)

**Image 3** - The Academy homepage.

The first thing that caught our eye was the “Register” link in the upper right corner.  Upon clicking the link, we were presented with a simple registration page.

![4](/images/academy/4.png)

**Image 4** - The Academy registration page.

One of the normal steps that we would take when looking at a webpage is to view the source.  In this instance, the source of the registration page had a hidden field which is a red flag for us.  The “roleid” field is something we should keep in the back of our minds.

![5](/images/academy/5.png)

**Image 5** - The Academy registration page source.

Another common test to run against a web server is a tool called nikto.

![6](/images/academy/6.png)

**Image 6** - The nikto command.

Nikto did give us a little bit of help, in the form of an admin page.

![7](/images/academy/7.png)

**Image 7** - The nikto results.

And sure enough, we have an admin login page.

![8](/images/academy/8.png)

**Image 8** - The admin login page.

Back at the registration page from Image 4, we are going to create an account.

![9](/images/academy/9.png)

**Image 9** - The Academy registration page.

The app lets us create an account and login!

![10](/images/academy/10.png)

**Image 9** - The Academy dashboard.

Next, we headed over to the admin login page to see if those same credentials would work.

![11](/images/academy/11.png)

**Image 11** - The Academy dashboard.

But unfortunately, they do not and the app kicks us back.

![12](/images/academy/12.png)

**Image 12** - The admin login page.

## Exploitation
Let’s head back over to the registration page from Image 4.  We are going to create a new user that we hope will become an admin.  We will manipulate that hidden field we saw in Image 5.

![13](/images/academy/13.png)

**Image 13** - The Academy registration page.

Before we click “Register”, we will fire up Burp to capture our request.

![14](/images/academy/14.png)

**Image 14** - The captured request.

Once we have captured the request, we will change the value of the “roleid” parameter to “1”.

![15](/images/academy/15.png)

**Image 15** - The captured request.

After forwarding the request on, it appears we were again successful in creating an account.

![16](/images/academy/16.png)

**Image 16** - The Academy dashboard.

But the real test here is going to be if we can login to that admin page from Image 12.

![17](/images/academy/17.png)

**Image 17** - The admin login page.

After click on “Login”, we find that yes, we can login as an admin!!!

![18](/images/academy/18.png)

**Image 18** - The admin dashboard.

It appears as though this is a to-do list for an administrator.  We can see completed tasks, but what catches our eyes is the “dev-staging-01.academy.htb” item at the very bottom of the list.

Let’s navigate to that page and see what we are presented with.  What we are given is a log file for a program called “laravel”.

![19](/images/academy/19.png)

**Image 19** - The dev-staging-01.academy.htb page.

Scrolling down on the page, we see that it gives us some information.  This information will prove to be useful in our attack.  Note the “APP_KEY” parameter.

![20](/images/academy/20.png)

**Image 20** - The dev-staging-01.academy.htb page.

Next, let’s see if there are any known exploits for laravel.  It appears that there is a Metasploit module.

![21](/images/academy/21.png)

**Image 21** - Searchsploit results.

First, we will fire up Metasploit...

![22](/images/academy/22.png)

**Image 22** - The “msfconsole” command.

In looking at the web entry for the exploit, we can see that it is pretty straightforward as Metasploit modules go.

![23](/images/academy/23.png)

**Image 23** - The Rapid7 entry for the exploit.

As per the searchsploit results from Image 21, we will use the below exploit in Metasploit.

![24](/images/academy/24.png)

**Image 24** - Selecting the appropriate exploit.

Next, we will show all of the available options for our exploit.

![25](/images/academy/25.png)

**Image 25** - Exploit options.

Notice that one of the options that Metasploit is asking for is the “APP_KEY”.  Conveniently, we know what the APP_KEY is from Image 20.  We will use the “set” command in Metasploit to set the parameter.

![26](/images/academy/26.png)

**Image 26** - Setting the APP_KEY.

After setting the APP_KEY, we will use the “set” command to set all of the other appropriate options.

![27](/images/academy/27.png)

**Image 27** - Exploit options.

Next, we will run our exploit and we now have a shell!!!

![28](/images/academy/28.png)

**Image 28** - A successful exploitation.

After using python to give us a more functional shell, we navigate the file structure to see where we “landed”.

![29](/images/academy/29.png)

**Image 29** - Exploring the directories.

In researching Laravel a little bit more, we found that it keeps its configuration in the “.env” file.

![30](/images/academy/30.png)

**Image 30** - Laravel documentation.

Let’s take a look at that file to see what else is in there.  It does not appear to be in our current directory.

![31](/images/academy/31.png)

**Image 31** - The contents of our current directory.

However, if we go up one directory, we find our .env file.

![32](/images/academy/32.png)

**Image 32** - The contents of our current directory with the .env file.

The contents of the .env file appears to be the same as what we saw on the admin dashboard back in Images 19 and 20.  So that isn’t much help.  

![33](/images/academy/33.png)

**Image 33** - The contents of our current directory with the .env file.

However, if we go up one more directory, we see that there are two “Academies” being hosted.  A dev one (which is what we have been interacting with) and another one that is maybe production.

![34](/images/academy/34.png)

**Image 34** - The contents of our current directory.

If we change into the “Academy” directory, we can see that there is another .env file.

![35](/images/academy/35.png)

**Image 35** - The contents of our current directory with the .env file.

In looking at the contents of this .env file, we discover a password!!!!

![36](/images/academy/36.png)

**Image 36** - The contents of the .env file.

In going to the home directory we can list out the available users on the system.

![37](/images/academy/37.png)

**Image 37** - The users on the Academy system.

In listing all of the files in the users home directories, the user called “cry0l1t3” had the user.txt flag in it, so I am guessing that is their password.  Let’s try to login as “cry0l1t3”.

![38](/images/academy/38.png)

**Image 38** - Logging in as “cry0l1t3”.

Success!  Now let’s get that user flag for our Hack The Box glory!

![39](/images/academy/39.png)

**Image 39** - The user flag.

## Post-Exploitation

Now that we have at least locked down a low privileged user account, we are ready to move on to root!

After some other snooping around, we checked which groups “cry0l1t3” belonged to.  We have found that they are in the “adm” group.

![40](/images/academy/40.png)

**Image 40** - Cry0l1t3’s groups.

Not knowing much about this group ourselves, we went to Google and quickly got our answer.

![41](/images/academy/41.png)

**Image 41** - Learning about the “adm” group.

As the search results suggested, we went to /var/log to poke around.

![42](/images/academy/42.png)

**Image 42** - The contents of /var/log.

There are a lot of files here to weed through, let’s see if we can narrow it down.

By listing the files with the -la flag, we can see owners and group accesses.

![43](/images/academy/43.png)

**Image 43** - The contents of /var/log.

But this is still a lot, let’s filter it down a bit by only looking at things that the adm group can get to.

![44](/images/academy/44.png)

**Image 44** - The contents of /var/log, filtered to the adm group.

After looking at the permissions, it looked like the “audit” directory might be fruitful.  We changed into the “audit” directory and took a look at the files.

![45](/images/academy/45.png)

**Image 45** - The contents of /var/log/audit.

In looking at these log files, there are different log types.  The first one that caught our eye was the “CRED_ACQ” type.

![46](/images/academy/46.png)

**Image 46** - The contents of one of the log files.

In order to figure out what this log type meant, we again went to Google.

![47](/images/academy/47.png)

**Image 47** - Learning about the “CRED_ACQ” log type.

Google gave us a great [resource](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security_guide/sec-audit_record_types) in which we could learn about all of the log types.

![48](/images/academy/48.png)

**Image 48** - Learning about the “CRED_ACQ” log type.

This particular log type did not give us much, but now we had a resource in which to learn about the others.  It did not take long until we came across the “TTY” log type, which could prove to be fruitful.

![49](/images/academy/49.png)

**Image 49** - Learning about the “TTY” log type.

In the “audit.log.3” file we found several TTY entries.  The parts of these entries appear to be hex encoded.

![50](/images/academy/50.png)

**Image 50** - The contents of “audit.log.3”

We found an online hex decoder and started to run our strings through the decoder.

![51](/images/academy/51.png)

**Image 51** - Decoding the hex.

It appears that the first line we tried, a user attempted to substitute mrb3n to run their commands.

![52](/images/academy/52.png)

**Image 52** - Decodin the hex.

In the second line, we discover mrb3n’s password.  This would make sense as the first thing the system would ask for when attempting to su is the password for the desired user.  When we try to “su” to mrb3n ourselves, sure enough, it works!

![53](/images/academy/53.png)

**Image 53** - Now running commands as mrb3n.

In our exploration of the mrb3n account, similar to how we investigated cry0l1t3, we listed mrb3n’s sudo rules.

![54](/images/academy/54.png)

**Image 54** - mrb3n’s sudo rules.

It appears that mrb3n can run composer with sudo.  One of the great resources for escalating privilege on linux is [gtfobins](https://gtfobins.github.io/).  Gtfobins is a collection of vulnerable binaries that can be used to escalate privilege amongst other tricks.  Conveniently, gtfobins has [an entry for composer](https://gtfobins.github.io/gtfobins/composer/#sudo).

![55](/images/academy/55.png)

**Image 55** - gtfobins entry for composer.

After we execute the commands from the gtfobins entry, we do in fact become the root user!!!

![56](/images/academy/56.png)

**Image 56** - We are root!

Lastly, we will grab the root flag for our Hack The Box glory.

![57](/images/academy/57.png)

**Image 57** - The root flag.

## Reporting
All good penetration tests have one thing in common: good reports.  This is a *very* simplified version of how I would write the 3 findings if I were submitting them.

### Finding 1 - Hidden field used to assign user privilege.
__Severity__: CVSSv3.1 AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:N/A:N - High to Critical

__Summary__: It was found that external attackers could create administrator level user accounts via a hidden field.  The attackers would then have access to the laravel admin page.  It is recommended that admin level privileges are assigned in a different industry standard method.

### Finding 2 - Clear text passwords in log and configuration files.
__Severity__: CVSSv3.1 AV:N/AC:L/PR:H/UI:N/S:C/C:H/I:N/A:N - High

__Summary__: It was found that multiple clear text passwords were being kept in log and configuration files.  The attackers would be able to escalate privilege and move laterally through the system.  It is recommended that an audit be completed for clear text passwords and any found passwords be changed and removed.

### Finding 3 - Poor sudo rules.
__Severity__: CVSSv3.1 AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H - Critical

__Summary__: It was found that a user could run the “composer” command with sudo.  An attacker could leverage this with publicly available code to escalate their privilege to root.  It is recommended that an audit be completed for sudo rules and any inappropriate rules be removed.

### Attack Chain
Nmap > Port 80 > Registration page with hidden field "roleid" > captured request in Burp > edited roleid to "1" > can now login to /admin.php as an admin > in the admin page we discovered, there is mention of "dev-staging-01.academy.htb" > dev-staging mentions a product called "laravel" > searchsploit reveals several vulns > looking at the metasploit modules, it needs an app_key, which is on the dev-staging page > set all of the options in Metasploit, exploit > shell!!! > found user "cry0l1t3" > there is a user.txt in their home directory > according to Laravel documentation, the config file is stored in ".env" > there appears to be a dev and production "academy" > there was a password in the production .env file > it was cry0l1t3's password > USER!!! > cryolite was in the "adm" group > this group can read log files in var/log/audit > reviewed 4 different log files and the types of logs (TYPE=) > found log entries for "TTY" > found hex encoded password for mrb3n > changed to mrb3n > mrb3n can run composer as root > there is a GTFOBins entry > root!

## Conclusion
I hope you enjoyed the write up and the system!  Feel free to reach out to me on [LinkedIn](https://www.linkedin.com/in/aaron-ragusa-3258ba174/) with any questions!

