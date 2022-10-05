---
layout: default
title : Ov3rkill and Secret Service
---

## Ov3rkill
![Screenshot_20221005_075453](https://user-images.githubusercontent.com/24994796/194021063-c84c9fcb-8897-43ce-8af7-cd6ace924649.png)

Ov3rkill was a misc challenge

The Challenge was about automationg basically, laying emphasis on your need to know about bash scripting or any scripting at 

So Downloading the the file we have a zip file, we have a assword protected zip file, cracking it, unzipping it we were greeted with another protected zip file 
From the description it says bash as far as the eye can see
Initially I thouht it'll be like 20-50 nested zip file so I went down the manual route because I had the commands in cycles but later I decided to admit defeat and create or modify a script for my need, initially I was using zip2john then crack the hashes with john but if i want to script that, it'll take a bit of time to get the script to properly work and in this life If you can make things easier why not do it

There's Another tool for cracking passworded zip file, it's called fcrackzip
So let's dive into it
![Screenshot_20221005_080429](https://user-images.githubusercontent.com/24994796/194021396-57c66a96-35cf-4438-8f3a-04f7d74732d9.png)

```
#!/usr/bin/env bash

while [ -e *.zip ]; do
  files=*.zip;
  for file in $files; do
    echo -n "Cracking ${file}… ";
    output="$(fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u *.zip | tr -d '\n')";
    password="${output/PASSWORD FOUND\!\!\!\!: pw == /}";
    if [ -z "${password}" ]; then
      echo "Failed to find password";
      break 2;
    fi;
    echo "Found password: \`${password}\`";
    unzip -q -P "${password}" "$file";
    rm "${file}";
  done;
done;

```

So basically what the script does is as long as there's a file endin with the .zip extention crack it, copy the password found, use it to unzip the file 
This file will only work once I could Iterate it to repeatedly loop and go to the new directory but that'll start getting a bit complex so I created an alias in my .zshrc to run the script as a command

![Screenshot_20221005_082838](https://user-images.githubusercontent.com/24994796/194021509-8961db83-c495-41e7-819a-6ca69411ba8c.png)


did a little in terminal bash scripting 
```
for i in {1..1000}; do k; cd ./*/;done
```
So what this means is do the following command 1000 times
In linux you can run multiple commands one after the other with `;` `&&`
and so many others while ipe `|` runs the command at the same time like both at once there's a difference between the two but checkout the man page of bash for more details

- `k` is our script command
- `cd ./*/` will change directory to the directory in the current directory `./` In linux * is a wildcard but since we know it's only one directory that get's created after unzipping it's safe to use
- `for i in {1..1000`} is basic looping telling us do this 1000 times

Initally the zip was nested 9k+ times but because so many peole pc couldn't handle the juice the shortened it to about 900+ 971 or so

![Screenshot_20221005_082957](https://user-images.githubusercontent.com/24994796/194021600-9d9c483f-9445-4f55-82e4-095d43e62d33.png)
![Screenshot_20221005_083026](https://user-images.githubusercontent.com/24994796/194021612-f58f6a6e-e39a-485f-8735-41c0f3629a7f.png)

Runnin that took about a 3 three minutes or less and we got to the file we needed
![Screenshot_20221005_084117](https://user-images.githubusercontent.com/24994796/194021711-78b6a9bb-b3e2-49af-b3e8-95beebdb800b.png)
Viewing the file we saw a cryptic looking text 
the file format is abcctf{}
![flag](https://user-images.githubusercontent.com/24994796/194021823-8a965775-46e4-47fe-9c06-c75e89a56f38.png)

The picture is hinting at using a script or thanking the fact a bash script got us here, scannin the text with google lens to copy out the ciphertext then sending it to dcode.fr/en to identify what sort of cipher it was

I got a few options such as caeser cipher, shift cipher, substituition, rot13 and the rest
luckily I did something similar for CS50x where I created a script to encode and decode caeser and substitution cipher so I had an Idea of what to do but couldn't imlent it via code so I went the manual way
we know the flag format is abcctf{} and the beginning of the text was lnpqiv so the lnpq made me know it was incremntal, I started counting from a, from a to l was 11 if we add 11 to a we get l (ascii table and normal abcd look it up) and it was 12 to get n from b so we just keep adding as we go
So i went to dcode.fr/en shift ciher and started doing it trial and error proved me correct
![Screenshot_20221004_002649](https://user-images.githubusercontent.com/24994796/194021949-dbafedb5-62bb-457c-a8b4-45583bbd7fac.png)

Now all we have to shift is only the alphabets nothing else but count all ascii characters (l was 11, n was 12) continue in that order
![Screenshot_20221005_085730](https://user-images.githubusercontent.com/24994796/194022066-066c275b-2af0-45e7-9cbb-161ec88b1135.png)
![Screenshot_20221005_085736](https://user-images.githubusercontent.com/24994796/194022099-6c120e4e-a6a4-4459-9901-e3714fbbd61c.png)

flag was `-   abcctf{y0u_g0t_th1s_f4r???_m4d_r3sp3ct!_b4sh_aaaaaaaall_th3_w4y}`

## Secret Service
![Screenshot_20221005_090758](https://user-images.githubusercontent.com/24994796/194022419-2358a6bc-6c69-4521-86df-400f6c3299da.png)

This was pretty easy and straight forward 
![Screenshot_20221005_090730](https://user-images.githubusercontent.com/24994796/194022457-69d70ec8-b410-4b42-9975-44394f62b7e5.png)

we check the file type and notice it is a KGB archive, we make a copy with the .kgb extention then just extract

<br> <br>
[Back To Home](../../index.md)
<br>
