---
layout: default
Title: Easy Web
---

# Easy Web
Could be simple could be hard, depends on you xD. Jk
It's nice and easy

![Screenshot_20220515_191929](https://user-images.githubusercontent.com/24994796/168485618-876eecbd-f5ed-4211-9495-c0b26b0599cb.png)

Heading on to the website, we notice something in the URL (bottom of the page)

![Screenshot_20220515_174208](https://user-images.githubusercontent.com/24994796/168485840-489d5940-b177-457f-96be-0b26bec53ae1.png)

Let's put this through burp or zap to intercept packets and see what's going on here
looking through the results of zap we the api field once again, let's try changing the id parameter

![Screenshot_20220515_175851](https://user-images.githubusercontent.com/24994796/168485966-17be8248-c02c-46bd-8094-8132c49a6a6c.png)

we notice there's a source for id

![Screenshot_20220515_193039](https://user-images.githubusercontent.com/24994796/168486070-69b8cee0-c3b1-45bf-b25f-88a25f69dbf8.png)

Reading everything about the source from Zap (or burp) you'll notice something bout gimme-flag giving flag and any other, telling you they know nothing you speaking of.

![Screenshot_20220515_180312](https://user-images.githubusercontent.com/24994796/168486322-7ba8b036-7b45-4712-a263-31a3997d4894.png)

And there's our flag 

<br> <br>
[Back To Home](../../index.md)
<br>
