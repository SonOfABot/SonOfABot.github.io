---
layout: default
title: HTB -- Cronos
---

# Cronos

![Screenshot_20220523_025703](https://user-images.githubusercontent.com/24994796/169724639-a8f10b74-5269-4020-8f14-d50c8830664b.png)

## This is quite the box, honestly I've forgotten about a passive recon tool such as dig, cos for one I don't use it everyday and all that but thanks to research I had o use it cos no, it made life so easy

## Enough chatty chatty more hacky hacky 🥵

# Enumeration

First things first, Enumeration - No matter what you're doing you can't do without proper recon
So let's fire up nmap to scan for ports to get open ports

![Screenshot_20220521_020908](https://user-images.githubusercontent.com/24994796/169725100-2978f6d3-7e6a-414e-a887-70a60005bded.png)

Running a quick nmap stealth scan and a udp scan shows us what ports are up and running
- Port 22,53 and 80 are open
- That's ssh, domain and http respectively

So enumerating for subdomains gets us an admin page (Using FFUF)

![Screenshot_20220521_020829](https://user-images.githubusercontent.com/24994796/169725777-2c8c7ab5-c6c4-422d-9e8d-51ea78685538.png)

(Using dig)

![Screenshot_20220521_020838](https://user-images.githubusercontent.com/24994796/169725828-b624b058-d43b-43e1-81c5-36706cbddc85.png)

Dig gives us way more information in this case

# Initial foothold

Heading to the admin page we are faced with a simple login page that's vulnerable to sqli login bypass

![Screenshot_20220521_021748](https://user-images.githubusercontent.com/24994796/169726054-c29bfa63-f428-4bb3-a3b2-ad6efa9627f7.png)

![Screenshot_20220521_021731](https://user-images.githubusercontent.com/24994796/169726082-cd2c0db9-c7cc-43cf-897f-707e7088d942.png)

Logging in we are greeted with a net tool, and it functions like a web terminal xD

In linux terminal you can run multiple commands with some seperators such as ; , && 
So with that knowledge, adding ; id should run the traceroute and id, one after the other
Let's set up a listener then run a one liner revshell from payloadallthings

![Screenshot_20220521_022024](https://user-images.githubusercontent.com/24994796/169726539-6176fd0b-5735-4f17-9e00-b8fc944bcc17.png)

Upon catching a shell we run id, ls 
id command in Linux is used to find out user and group names and numeric ID's (UID or group ID) of the current user or any other user in the server. 
ls id used to list, lists files in a directory

Sometimes the config file contains password of a user but in our case it doesn't

On ![Screenshot_20220521_022616](https://user-images.githubusercontent.com/24994796/169726980-78dfefdc-762f-4dc4-b70d-e155795f1173.png)

navigating the machine we see we can access the user flag with our current user


# Privilege Escalation

After a series of enumeration we find out there's a PHP cronjob running every minute

![Screenshot_20220521_034128](https://user-images.githubusercontent.com/24994796/169726997-b5bb9508-5cf5-40e7-b735-2872d69bab43.png)

Reading the script that runs every minute we see it runs other scripts too

![Screenshot_20220521_034209](https://user-images.githubusercontent.com/24994796/169727156-13e8b518-bb5e-45f7-95ff-350a5397c45e.png)

And the file it runs as root is editable by us so now what we do is, get a php rev shell to the machine then use it to replace the contents on app.php

![Screenshot_20220521_034258](https://user-images.githubusercontent.com/24994796/169727343-1c5f4f7e-c749-4209-b065-cef43347921f.png)

Run up a listener and wait for a minute and we are root


And we done

<br> <br>
[Back Home](../../index.md)
<br>
