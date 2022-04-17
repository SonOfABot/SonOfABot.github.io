---
layout: default
title : TIMELAPSE - HTB
---

![Screenshot_20220415_082046](https://user-images.githubusercontent.com/24994796/163691386-f35f7eef-3eec-440d-9943-3a553f0acde3.png)

# I Can't Even Say Less Chatting More Hacking Because Nah This Box Deserves Some Respect, Really Educational 🥸
I Advice You Give It a Go First Before Following The Steps Here There are Different Ways Of Getting Root, Two I Know Of 😎


# So no futher ado let's jump right into it 💨

Firing Up The Machine We Get An Ip address

# ENUMERATION 🔍

![Screenshot_20220416_231838](https://user-images.githubusercontent.com/24994796/163691661-31605373-f192-49a3-ba5b-541338be41f9.png)

- So many posts are open I cant's start listing them but yh we'll focus on the mains
- Port 139, 445 is Open that can be connected to with smbclient 
  - smbclient -L \\IP-Addy
 
 
 ![Screenshot_20220415_075948](https://user-images.githubusercontent.com/24994796/163691930-b48a20df-6e1e-4e76-8629-0ff0643fb335.png)

  - One share allows anonymous login, Connect to it and get the files
  - What's the file type ? 
  - What's the file used for ?
  - Can the file password be cracked ?
  - What's In the zip you cracked
  - What's the file ? Can it also be cracked ?
 
 ![Screenshot_20220415_075930](https://user-images.githubusercontent.com/24994796/163691877-da7d3120-8d6b-49b2-8402-a2efc343ca97.png)

While cracking the pfx file if it shows no hass loaded there's a slieght issue with your hash, remove any ', "b' "

![Screenshot_20220415_080005](https://user-images.githubusercontent.com/24994796/163691998-e5a6da30-6344-4dcd-ab5d-ee63a2056cdb.png)

Extract the pfx file
- openssl pkcs12 -in [yourfile.pfx] -nocerts -out [drlive.key]
- openssl pkcs12 -in [yourfile.pfx] -clcerts -nokeys -out [drlive.crt]
- openssl rsa -in [drlive.key] -out [drlive-decrypted.key]

![Screenshot_20220415_080030](https://user-images.githubusercontent.com/24994796/163692052-88cbdfe2-f74c-41d4-9ac7-976eb704f203.png)

# Initial Foothold

Remember the zip's name ? winrm ?
Login via evil-winrm

![Screenshot_20220415_080039](https://user-images.githubusercontent.com/24994796/163692090-245f1479-cc65-488c-b1fe-7e65d01b8664.png)

So We're in

![gif](https://thumbs.gfycat.com/LightheartedObviousBlowfish-max-1mb.gif)


You can run any enumeration script of your choice 
just run upload "script" (make sure it's in the directory you're running evil-winrm from)

![Screenshot_20220415_080039](https://user-images.githubusercontent.com/24994796/163694170-ec67d184-55bb-463c-8daa-92d3496b2500.png)

run your sript, you'll notice the powershell history is there, view it

![Screenshot_20220415_080104](https://user-images.githubusercontent.com/24994796/163694199-9317a356-db73-4215-b51d-30d25d6f9036.png)

So we can see a series of instructions

![gif](https://y.yarn.co/b512fbb4-9f85-45d0-bdf6-edbba4d22f3e_text.gif)

We repeat the commands, and we're able to successfully act as the svc_decoy user

![Screenshot_20220415_080129](https://user-images.githubusercontent.com/24994796/163694374-c8f83a01-91a2-47c4-af90-4ea3279ef4cb.png)

![git](https://y.yarn.co/00f98e27-6797-4f66-ae7b-6e1ed87670cf_text.gif)

# Privilege Escalation

So we see now have to think how to get root after all we have some credentials already
Remeber in the smbclient folder there was a folder of Help blah blah it contained things about ldaps and we from our scans we have ldap3 open ports
So we dump ldaps

![laps python dumping script](https://github.com/n00py/LAPSDumper)

![Screenshot_20220415_075854](https://user-images.githubusercontent.com/24994796/163694505-ae5400c3-262b-43d6-ac5f-eb7af8740142.png)

We have admin password

So now you do understand
![gif](https://media0.giphy.com/media/xUNd9L2VQyFALRuJdC/giphy.gif)

We have to change the commands in history
![gif](https://media0.giphy.com/media/3EiExzquN7mzTCVK3m/giphy.gif)

Copy the password from that dump (my pass is different cos i had to reset the machine half way)

![Screenshot_20220415_080154](https://user-images.githubusercontent.com/24994796/163694649-2878ce0a-3277-4d1b-9073-5b9c46ccf9ca.png)

Great we can run actions as admin
we upload nc.exe through our evil-winrm
Turn off win defend or whitelist the upload path (c:\users)
![Screenshot_20220415_080216](https://user-images.githubusercontent.com/24994796/163694772-e6b3547f-02d3-41f2-bd55-291e8ce6022e.png)

run netcat revshell

![Screenshot_20220415_080255(1)](https://user-images.githubusercontent.com/24994796/163695189-2f6d5ab6-a9c5-4963-a6b7-30e9e76063ea.png)

on your attacking machine you'll catch a shell

![gif](https://slack-imgs.com/?c=1&o1=ro&url=https%3A%2F%2Fthumbs.gfycat.com%2FDefiantNeedyBrownbutterfly-size_restricted.gif)

![gif](https://memegenerator.net/img/instances/47400398/im-in.jpg)

![Screenshot_20220415_081018](https://user-images.githubusercontent.com/24994796/163694809-dd9ca1b2-1949-4bb4-bb0a-bcbfa8f79af3.png)

![gif](https://c.tenor.com/A21UNwQmEuAAAAAd/hacker.gif)

# AND WE ARE DONE
LOOK FOR FILES ON YOUR OWN, it's your little task

Hit me up on [Twitter](https://twitter.com/abdulmalik_ttg) if you run into any issues



<br> <br>
[Back To Home](../../index.md)
<br>


