---
layout: default
title: Hacker
---

## **Hacker**

![Screenshot_20221005_235811](https://user-images.githubusercontent.com/24994796/194181938-ea85d56f-61f3-47b5-ad02-179dd28550cc.png)

Hacker Sounds badass right ? yep it is but this was quite easy
Heading onto the website we were greeted with a website and all it does is to upload a file

![Screenshot_20221005_173656](https://user-images.githubusercontent.com/24994796/194182019-8060d0fd-58e3-4518-9628-b15a3137c522.png)

`File upload vulnerabilities are when a web server allows users to upload files to its filesystem without sufficiently validating things like their name, type, contents, or size.`
Read more here ![file upload vulnerability](https://portswigger.net/web-security/file-upload)

![Screenshot_20221005_173759](https://user-images.githubusercontent.com/24994796/194182074-6094c56f-3981-488f-b730-c337a5ec73a7.png)

Here there are filters such as file type and size
- it must be a a jpeg image 
- it must be 35 bytes or less
Bypassing this was easy we simply craft a really small php payload prepend it with a number of A's then using hexedit we change the hex character, for jpeg files the magic byte is `FFD8`, I prepended 4 A's in Hex A is 41 (remember your ascii table)

Let's dive into it
my payload was really tiny(typing it out will mess up my md file)

![Screenshot_20221005_173857](https://user-images.githubusercontent.com/24994796/194182147-2fa22936-001c-4c15-8218-214623855e35.png)

`AAAA<?=`$_GET[0]`?>`

saving that and then checking the file type we see it's ascii text

![Screenshot_20221005_174149](https://user-images.githubusercontent.com/24994796/194182573-e13747b6-cae5-4827-999a-6b0f26edbe3a.png)
![Screenshot_20221005_174247](https://user-images.githubusercontent.com/24994796/194182579-7ec02b0a-2069-481b-8cf7-e5aee573cc08.png)

Now we edit the hex character to that of a jpeg

![Screenshot_20221005_173945](https://user-images.githubusercontent.com/24994796/194182643-c2f3a457-685f-441a-896a-0bbe1bc800e4.png)
![Screenshot_20221005_174308](https://user-images.githubusercontent.com/24994796/194182648-a225f9c6-2f9b-4858-9234-6913bdc3fc40.png)

checking the file type again we see it has changed

![Screenshot_20221005_174317](https://user-images.githubusercontent.com/24994796/194182704-38fc5dc2-1867-4a11-8365-4dabff2ce2a6.png)
![Screenshot_20221005_174408](https://user-images.githubusercontent.com/24994796/194182708-cd4268ca-01d3-47ff-9945-9487b9823488.png)

So now we upload our shell

![Screenshot_20221005_175022](https://user-images.githubusercontent.com/24994796/194182765-b676f61a-f9a9-4b57-bf5d-d52da800ce4a.png)

While it's possible to guess the file path running a directory fuzzing helps know the files and directories on the web server

![Screenshot_20221005_175800](https://user-images.githubusercontent.com/24994796/194182822-bb4a7a3e-09ea-4354-8a4d-7890fdd31e97.png)
![Screenshot_20221005_235955](https://user-images.githubusercontent.com/24994796/194182830-16578047-8d24-4a21-8013-0dac2ddd9f2d.png)

running curl does the same thing 

![Screenshot_20221006_001130](https://user-images.githubusercontent.com/24994796/194182855-01c8345a-c3ee-4a5f-ae6e-f1e6cec51fa8.png)


It did take a while finding our flag but yh we did find it in upload.php

![Screenshot_20221005_235542](https://user-images.githubusercontent.com/24994796/194182893-954e1232-7978-4d5d-812b-70dd20aa2359.png)

So that's that

<br> <br>
[Back To Home](../../index.md)
<br>
