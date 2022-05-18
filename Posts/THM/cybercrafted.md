---
layout: default
Title: Cybercrafted -- TryHackMe
---


# Cybercrafted
![Screenshot_20220518_204314](https://user-images.githubusercontent.com/24994796/169122394-6c2d46c9-f543-4ad8-ae79-ce1714dd58a6.png)

## Okay No This Was A Very Interesting Room, Had Lot's Of Interesting Ideas And If You're Not Careful, You'll Dig 😑
So Yh Let's Jump Right Into It

![gif](https://c.tenor.com/j3WSiehLt2AAAAAM/portal-jumps-into-a-random-portal.gif)

## Enumeration
First Things First Enumeration

Let's run a port scan to know services that are running on the open ports if any

Running a full portscan while enumerating services will do just fine for our needs

```nmap -sVC -A -T4 --min-rate 500 -p- -Pn -oA Fullfastscan Machine IP```

![Screenshot_20220518_205516](https://user-images.githubusercontent.com/24994796/169132700-c3dac022-f71b-4898-b4fb-3979b37e3d91.png)

After the scan we get 3 ports open
- Port 22, 80, 25565
- Respectively SSH HTTP and Yh you guessed it genius Minecraft server 🥸

So Let's Run FFUF to list directories and sub-domains (and it's directories)

Locating sub-directories is not really tricky just you have to filter by size to get rid of silly things (using ffuf)

``` ffuf -w <path-wordlist> -u https://test-url/ -H "Host: FUZZ.site.com" ```

![Screenshot_20220518_031839](https://user-images.githubusercontent.com/24994796/169140639-d888deb6-9786-4a9f-b659-57943d749e2a.png)

On The Admin Sub-domain We see the existence of a login page and on the Store sub-domain there's a search.php 
So Basically What It Does Is Simply Search For Items, In-game Items So yh We can make a safe guess It's connected to database after trying a series of sqli and it didn't work why don't we take it a big dog SQLMAP

This Part Was A Bit Tricky For Me, The URL Refused To Work For Some Reason So I used Burp To Intercept The Request Then Copy The Intercepted Request To A File 

![Screenshot_20220518_203319](https://user-images.githubusercontent.com/24994796/169142218-3e83586f-20d8-4b22-b63c-510b76cb941c.png)

![Screenshot_20220518_203403](https://user-images.githubusercontent.com/24994796/169142257-d781874c-223b-4178-ab7a-e8ffde3604f8.png)

![Screenshot_20220518_203642](https://user-images.githubusercontent.com/24994796/169142326-fe5af78f-4611-4e8e-9165-5ca533cac806.png)

- I didn't Really Dump All I Just Let It Ran Till I Had Enough Info
- Then When I Did I Changed My Request

![Screenshot_20220518_203854](https://user-images.githubusercontent.com/24994796/169142663-9a3862e1-a999-4c6e-ac29-5ee9ae5214d3.png)

![Screenshot_20220518_203814](https://user-images.githubusercontent.com/24994796/169142693-f491847e-d994-437b-a414-588177bedcfb.png)

Now We Have
- User
- Password Hash
- A Web Flag

## Initial Foothold

Identify The Hash With Hashid, Decode With John Get Password
Logging Into The Admin Panel We Can Execute Commands, Sweet 🤤
Get A RevShell running

![Screenshot_20220518_042309](https://user-images.githubusercontent.com/24994796/169144639-cf4debe7-7145-422e-9ac7-17f9037a4d41.png)

## Privilege Escalation

In Our Scans Port 22 Was Open So Yh There Should Be SSH keys Lying Around After All Hackers Don't Break in, We login 😎

Heading over to the user xx blah blah home directory we see SSH folder and some files 

Let's Copy The Id_rsa file to our system 

changing the id_rsa mod and trying to login doesn't seem to work 
It seems to be encrypted
John Seems be an indispensible tool in this room huh
We run ssh2john then crack the hash

![Screenshot_20220518_044646](https://user-images.githubusercontent.com/24994796/169145870-4b3ff6c6-5740-4de2-acc8-034897763743.png)

We're In 
Locating Folders we can (access) read or Write We see a folder in opt and some sus plugin
Viewing the log shows us password in cleartext 

SSH to cybercrafted account

![Screenshot_20220518_051016](https://user-images.githubusercontent.com/24994796/169146306-b7258bb2-c266-4891-8ccf-80c6e3831271.png)

Running sudo -l we see cybercrafted can run ``` sudo /usr/bin/screen -r cybercrafted```
Which gives us screen with root privileges

Holding Ctrl+a c - Creates a new window (shell) so yh we golden 

![Screenshot_20220518_061445](https://user-images.githubusercontent.com/24994796/169146909-2b68be9c-de6b-44d1-806d-ac56e86a38cd.png)

That's It for Cybercrafted Hope you Learnt A thing or at Least Had Fun

Hit me up on [Twitter](https://twitter.com/abdulmalik_ttg) if you run into any issues or want me to change my writeup style



<br> <br>
[Back To Home](../../index.md)
<br>
