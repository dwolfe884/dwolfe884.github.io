---
title: eXcellent encryption
date: 2021-11-09 16:30:00 +/-1111
author: David Wolfe
categories: [Writeups, sp00kyCTF]
tags: [ctf,writeup,sp00ky,ISU] 
---
## Description:
```
A witch put a spell on my flag! Can you help break this and get me my flag back so I can trade it for candy?

Flag format: sp00ky{
```
This was a challenge I created for the Information Assurance Student Group's anual sp00ky CTF

## Given information
For this challenge players were given the hex bytes `00 1b 49 5a 0a 1a 10 14 39 1c 2b 15 3c 21 43 09 26 5a 0f 0d 05 2c 38 49 06 37 0a 05 14 34 14 33 3e 05 22 01 18 2d 35 22 0b 0a 1f 07 3c 04 06 06 16` and the title of the challenge indicated that this is some form of XOR encryption. On top of this the player is also told that the plain text for this XORed string of bytes is the flag.

## My Approach
With this information I expected people to use a known plaintext attack against the string and recover the key needed to decrypt it. You can perform this attack by attempting to XOR the encrypted string with as much of the known plaintext as possible.

![Normal](/images/eXcellent/recip.png)

The above cyberchef recipe accomplishes this goal. Since XOR works in both directions, `IF [A] (XOR) [B] = [C] THEN [C] (XOR) [A] = [B]`. In this example `A` would be the flag, `B` would be the key, and `C` would be our cipher text. Because the plaintext is a flag, we know it must start with the characters `sp00ky{`. If we know that into cyberchef and give the xored bytes in as input we get the following output:

![Normal](/images/eXcellent/chef.png)

Since we only know 7 characters of the plain text we should only look at the first 7 characters in the output. This spells out `skyjack` which should be used at the key for the original plaintext. Replacing our known plaintext with the key we just extracted gives us the flag.

![Normal](/images/eXcellent/solved.png)

## Solved

```sp00ky{gReAt_J0b_0nnn_S0lVing_mY_fIrsT_ChallEnge}```

## Final Thoughts

This was my first attempt at creating a CTF challenge. I had some fun, but I think I made this too difficult for the original CTF it was intended for. I think there were only 1 or 2 teams that ended up solving it. BUT, it was worth more than most other challenges so I guess it evens out. Sp00ky CTF was a ton of fun and I hope everyone learned something!