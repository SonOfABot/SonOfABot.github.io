![[Screenshot_20221005_235811.png]]
Hacker Sounds badass right ? nope this was quite easy 
Heading onto the website we were greeted with a website and all it does is to upload a file 
![[Screenshot_20221005_173656.png]]
`File upload vulnerabilities are when a web server allows users to upload files to its filesystem without sufficiently validating things like their name, type, contents, or size.

Read more here ![link](https://portswigger.net/web-security/file-upload)
![[Screenshot_20221005_173759.png]]
Here there are filters such as file type and size
- it must be a a jpeg image 
- it must be 35 bytes or less
Bypassing this was easy we simply craft a really small php payload prepend it with a number of A's then using hexedit we change the hex character, for jpeg files the magic byte is `FFD8`, I prepended 4 A's in Hex A is 41 (remember your ascii table)

Let's dive into it
my payload was really tiny (typing it out will mess up my md file)
![[Screenshot_20221005_173857.png]]
saving that and then checking the file type we see it's ascii text
![[Screenshot_20221005_174247.png]]
![[Screenshot_20221005_174149.png]]
Now we edit the hex character to that of a jpeg
![[Screenshot_20221005_174308.png]]
![[Screenshot_20221005_173945.png]]
checking the file type again we see it has changed
![[Screenshot_20221005_174317.png]]
![[Screenshot_20221005_174408.png]]

So now we upload our shell
![[Screenshot_20221005_175022.png]]
While it's possible to guess the file path running a directory fuzzing helps know the files and directories on the web server
![[Screenshot_20221005_235955.png]]
![[Screenshot_20221005_175800.png]]
running curl does the same thing 
![[Screenshot_20221006_001130.png]]

It did take a while finding our flag but yh we did find it in upload.php
![[Screenshot_20221005_235542.png]]

So that's that