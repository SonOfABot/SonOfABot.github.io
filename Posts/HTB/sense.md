---
layout: default
title: SENSE
---

# HTB -- SENSE
This is a retired htb box, quite intersting and direct

![Screenshot_20220523_005522](https://user-images.githubusercontent.com/24994796/169747552-7f0e9bf2-b35b-4ddb-a27c-e5316db79441.png)

Let's Just Jump Right Into It

# Recon 
Running NMAP We Get Two Open Ports 80, 443; HTTP and HTTPS respectively

![Screenshot_20220523_074908](https://user-images.githubusercontent.com/24994796/169753063-7771753b-cacb-402f-85b4-d962e7e2b6b0.png)

Fuzzing for directories on the https with ffuf gives us a list of directories

I'll advice you to let the entire wordlist runthrough so you don't miss crucial things

![Screenshot_20220523_080220](https://user-images.githubusercontent.com/24994796/169753485-caa1aa27-144d-4697-b028-a30bdf3be2da.png)

This box taught me you really have to run 1-3 enumeration of a certain service to prevent diving head first at rabbit holes
Took close to 2 hours + enumerating and rabbit hole chasing
All because I used the wrong wordlist and set some silly extentions initally


# Foothold 

![Screenshot_20220522_014447](https://user-images.githubusercontent.com/24994796/169757838-ad14b81c-566b-4b82-9701-06869ae65712.png)

After getting the credentials from the system-users.txt we keep searching online then we find this

![Screenshot_20220522_225031](https://user-images.githubusercontent.com/24994796/169757580-210961d1-07ad-4ea7-bd07-c9250af38fcd.png)

So our login credentials should be ```rohit:pfsense```

Logging in we are greeted with this, seems the repid7 link was spot on with it's exploit 

![Screenshot_20220522_014456](https://user-images.githubusercontent.com/24994796/169758238-c072c3db-0e8f-4cdc-bdd8-0a218f4bcef4.png)

# Exploit

So here's the direct thing, once you exploit the vulnerability you get ROOT!

So yh I saw an exploit on exploitdb 

![Screenshot_20220522_225231](https://user-images.githubusercontent.com/24994796/169758507-e97367e1-2318-4795-8df6-0b2f7e60c12b.png)

Copy/Download the python code 
Read it, try to understand it a bit
Set up your listener
Execute it

![Screenshot_20220523_010936](https://user-images.githubusercontent.com/24994796/169759964-73c61352-11c2-406f-afe2-481a141b2d0d.png)

![Screenshot_20220523_010931](https://user-images.githubusercontent.com/24994796/169761888-4c658584-432b-43ac-8613-899be8ede646.png)

# AND WE ARE DONE

Hit me up on [Twitter](https://twitter.com/abdulmalik_ttg) if you run into any issues



<br> <br>
[Back To Home](../../index.md)
<br>



