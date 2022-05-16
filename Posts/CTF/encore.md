---
layout: default
Title: Digital Forensic - Encore
---

# Digital Forensic - Encore
The only digital forensic in the CTF
![Screenshot_20220516_045840](https://user-images.githubusercontent.com/24994796/168512658-4cd2bcbd-e9fb-4cf1-8bbc-604010b9c6c8.png)


It was quite Intersting

Pls note stegsnow may not extract the flag for you considering it works on differently on different OS

I did not have this issue considering Parrot is OP 😎

# Solution 

First I used binwalk to extract the protected.jpg incase anything was embed in it

Binwalk is a tool for searching a given binary image for embedded files and executable code. Specifically, it is designed for identifying files and code embedded inside of firmware images.
![Screenshot_20220516_043826](https://user-images.githubusercontent.com/24994796/168512925-bced99a1-fb66-40f2-8512-fd7ca296dc61.png)

## Now Go To The Extracted Folder

![Screenshot_20220516_044325](https://user-images.githubusercontent.com/24994796/168513243-0209ff9c-9a5b-4bd8-8825-4edb908518f3.png)

Viewing the =.txt file gave us a hint and password 

We just stegsnow it 🥶
Stegsnow is a program for concealing messages in text files by appending tabs and spaces on the end of lines, and for extracting messages from files containing hidden messages. Tabs and spaces are invisible to most text viewers, hence the steganographic nature of this encoding scheme.

And That's A Wrap !!!

<br> <br>
[Back Home](../../index.md)
<br>

