---
layout: default
Title: Bella Crypter
---

# Bella Crypter Was A Fun Challenge

![Screenshot_20220515_155553](https://user-images.githubusercontent.com/24994796/168476635-6ea6c087-d80a-475f-ba69-da0380729c5f.png)

- Tools Used Hashcat/JohnTheRipper, Hashid, modified rockyou.txt wordlist and my secret weapon GOOGLE
  
Let's get Into it 

- First the given hash was identified with hashid 
![Screenshot_20220515_154201](https://user-images.githubusercontent.com/24994796/168476789-c804fad6-c403-4ab5-b3bd-8f39ef6d9229.png)

Bcrypt, was the gotten hashid

We can make a quick research bout decoding bcrypt hashes if you will, It'll enhance your understanding

# Using hashcat 
Save the hash to a file
```echo "$2y$12$dwt1bzj6pcyc3dy1fwz5ieeuznr71eenkjkulyptsgbx1h68wsrom" > hash```
``` hashcat -m 3200 hash rockyou.txt```

So now here's the catch decrypting bcrypt generally takes time and I for one think it's not really nice giving them in a ctf unless they have the time or hont on maybe decoded string length yuh zimmi ??

What's 4 days 😹😹😹
In truth it took around 5 hours or so 

# Google
Here's Where Google Comes in
Looking up the hash I saw It's same as the one on THM
Now I Recall Solving something similar 
![Screenshot_20220515_164353](https://user-images.githubusercontent.com/24994796/168479006-cec5f26f-6a5b-4183-901b-63e04ed29fbe.png)

We grep letter words from rockyou.txt
![Screenshot_20220515_165536](https://user-images.githubusercontent.com/24994796/168479437-e2abf19a-8fc6-4f52-972a-98b419fa9119.png)

Run hash cat again
![Screenshot_20220515_165725](https://user-images.githubusercontent.com/24994796/168479457-2773c299-562a-4231-80cb-8f4a3244d181.png)
Seems better now 😹

So now we wait

We got bleh as the decoded string

We done

<br> <br>
[Back To Home](../../index.md)
<br>


