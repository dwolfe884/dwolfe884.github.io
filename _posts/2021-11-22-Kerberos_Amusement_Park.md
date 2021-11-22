---
title: Kerberos Amusement Park
date: 2021-11-22 9:00:00 +/-1111
author: David Wolfe
categories: [Writeups, IASG]
tags: [presentation,writeup,ISU] 
---

## Overview
This is a quick blog post to discuss the talk I gave to Iowa State's Information Assurance Student Group (IASG). The topic for my presentation was `Intro to Kerberos`. I've recently gone through a number of different interviews in which I was asked for specifics of the Kerberos protocol and walk through a couple basic attacks. My goal for this presentation was two fold:
1. Students should leave with a basic understanding of Kerberos and be prepared for common interview questions.
2. To sure up my understanding of Kerberos by fielding questions and comments from the group during my presentation. 

## The Presentation
I've attached my slides to the bottom of this blog post if you want to follow along. The crux of my presentation rested on the analagy of Kerberos to an Amusement Park. I described the `KDC` as a ticket booth at the entrance of the park. In order to enter the park (domain) you need to provide credentials to the KDC and get a Ticket Granting Ticket (TGT) to enter the park. From there on out your TGT acts as your identity inside of the park. After you're inside of the park you see an amazing ride, The File Server Go-Round! You want to ride it so you go up to the ride and ask to get on. The ride operator would be visably confused as you try to give them your TGT. The ride operator would then explain that you need to go back to the ticket booth and use your TGT to get a specific Ticket for the File Server Ride. That's when you go back to the KDC and present your TGT along with the name of the ride you want to go on. The KDC will give a fancy new Ticket called a Ticket Granting Service (TGS) Ticket. Now that you've got your ticket for the File Server ride you go back to the ride and present the TGS. The ride operator will check to make sure you have the correct permissions. If everything checks out you will get a session with that service and be allowed to ride!

After I walked through this analogy I moved on to explain a couple of common attacks. The attacks I described were Golden Ticket, Silver TIcket, and Kerberoasting.

#### Golden Ticket
A golden ticket is simply a forged TGT. It requirs the attacker to have the password hash of the KRBTGT service account which lives on the KDC. With a golden ticket the attacker is able to create a TGS for any service with any level of permissions since they will be able to sign these forged tickets with the password hash.

#### Silver Ticket
A silver ticket is a forged TGS. It requirs the attacker to have the password hash of the service account that the specific service they want to access is running under. With a silver ticket an attacker has a much more limited scope compaired to a golden ticket. A silver ticket gives unlimited access only to the specific service that that ticket refers to.

#### Kerberoasting
Kerberoasting is one of the most common attacks in the wild and something that really wanted people to understand from my presentation. To go along with my Amusement Park analogy from above. Kerberoasting would be asking to ride every ride in the park at once in order to collect a large number of TGS Tickets. With these tickets it's possible to use a program such as `Mimicatz` to extract the password hash of the service account associated with each of the TGS Tickets and then run those hashes through `JohnTheRipper` to crack the hash and retrieve a plain text password. This style of attack is used for lateral movement and privilege escalation inside of a windows domain.

## Q&A

After my presentation there were a number of very thoughtful questions that I was able to answer and clear up. One question that came up that I wasn't immediantly able to answer was this,

`If you can use Kerberoasting to extract the password hash of a service account from a TGS... can't we just use that to immediatly create a Silver Ticket and have access to everything on that service?`

This was an excelent point and something I didn't realise was an issue until I presented these two attacks back to back. Because I didn't have a good answer off the top of my head, and we were at the end of the presentation, I decided to just start googling and having a more freeform discussion with the group. We were able to find two really useful articles that, when combined, showed us the full picture of Kerberos and why you aren't able to use Kerberoasting to go directly into a Silver Ticket Attack. In this [article](https://www.hackingarticles.in/deep-dive-into-kerberoasting-attack/) we found an image that showed exactly what was pulled out of a TGS when a successful Kerberoasting attack is performed.

![Hash pulled from TGS](https://1.bp.blogspot.com/-ecfq59EnyUU/XrHWZQ186pI/AAAAAAAAj4I/Kse_y11qUXY-hC0xUMGm4-CWh5mql3msgCLcBGAsYHQ/s1600/9.png)

From this we can see that the format for this hash is completely different than what we're used to. This hash is very long and takes in a lareg amount of metadata such as the computer name, domain name, username, and the name of the service tring to be accessed. We later found a different resource that confirmed this hash format to be in the 'KRB5ASRESP' format. With this in mind we momved on to the second article we found. [Here](https://www.tarlogic.com/blog/how-kerberos-works/) we saw a very useful diagram that showed exactly what was being sent back in a TGS.

![TGS Diagram](https://www.tarlogic.com/wp-content/uploads/2019/02/KRB_TGS_REP.png)

After reviewing this diagram we realized that in no way was the actual NTLM password hash of the service accuont being sent in the ticket. We combined this insight with our previous discussion on hash format and came to the following conclusion:

'What is exactracted from a TGS is something SIMILAR to the NTLM password hash of the service account, however it is not explicitly the password hash. Instead it is a different format that is derived from a combination of the secret key sent in the TGS a large amount of metadata. If you want to generate a Silver Ticket after performing Kerberoasting you would need to crack the KRB5ASRESP hash and then hash the plaintext password into the NTLM format.'

This satisfied the group and nobody else was able to poke holes in this theory.

## Final Thoughts

This was a fantastic opportunity. I was able to teach a group of students some very useful skills for their upcoming technical interviews as well as learn something myself. I believe I speek for everybody in the group when I say we learn best a group. It's much easier to ask questions and call people out on something they say if you are friends with them. That's why this presentation and the Q&A afterwards were so beneficial. We were able to use a sudi socratic method to come to the most correct solution as far as we couild tell. I plan on giving omre presentations to IASG in the future and I hope they will be as engaging as this one was! 

## Presentation PDF
![Normal](/images/kerberos/slide1.png)
![Normal](/images/kerberos/slide2.png)
![Normal](/images/kerberos/slide3.png)
![Normal](/images/kerberos/slide4.png)
![Normal](/images/kerberos/slide5.png)
![Normal](/images/kerberos/slide6.png)
![Normal](/images/kerberos/slide7.png) 