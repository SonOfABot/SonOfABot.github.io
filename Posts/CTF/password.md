---
layout: default
Title: Rev -- Password Verifier
---

# Password Verifier
![Screenshot_20220516_041638](https://user-images.githubusercontent.com/24994796/168509334-1613f6da-2540-4754-844d-9eae56436646.png)

This is a Rev challenge the only one in the competition
I'm new to Reverse Engineering but this one was pretty easy 

# Solution
First I unpacked the binary using upx 
UPX (Ultimate Packer for Executables)  is an open source executable packer supporting a number of file formats from different operating systems.
It is used for compressing binaries too.
So let's unpack first
![Screenshot_20220510_104917](https://user-images.githubusercontent.com/24994796/168509611-1c46e490-c260-424b-8a0c-146442cfea70.png)

seems like that was successful let's grep for CYSEC our flag format and see if there's anything

![Screenshot_20220510_104930](https://user-images.githubusercontent.com/24994796/168509730-299cf752-f02d-430f-9b42-d0149f03165a.png)


That's a Wrap We Done

<br> <br>
[Back Home](../../index.md)
<br>
