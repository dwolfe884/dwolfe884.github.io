---
title: Dodgy Databases
date: 2021-08-17 9:00:00 +/-1111
author: David Wolfe
categories: [Writeups, RACTF]
tags: [ctf,writeup,ractf] 
---
## Description:
```
One of our most senior engineers wrote this database code, it's super well commented code, 
but it does seem like they have a bit of a god complex. See if you can help them out.
```
We are give the source code for a simple service running that just takes in a username and adds that username to some backend database.
## Methodology
This challenge was under the pwn/reversing category. I first looked through the provided .c file for any obvious issues. The first noteworthy line of code I could find was on line 204.

![Free Vulnerability](/images/database/freeVuln.PNG)

In this snippet we can see the memory holding the admin object being freed and then immediately creating a new user object after that. This sequence of events causes a ```Use-After-Free``` vulnerability since the now freed admin object is still being passed into the ```users_register_user``` function. On top of that, because these objects are the same size and structure, it's very likely that our new user object would reuse some/all of the memory originally allocated for the admin object.

I continued to read through this source code and found our goal, in this case where the flag is printed to the screen.
![Goal](/images/database/goal.PNG)
This function shows a simple check for if the passed in admin object has a role of ```ROLE_GOD```. If it does, we get the flag.

With that static analysis out of the way, I moved on to a more dynamic approach. I launched the compiled binary with GDB and set a breakpoint at the ```users_register_user``` function. After entering a random username we hit the breakpoint and are able to see the exact instruction where this comparison for ```ROLE_GOD``` is taking place.

![GBD Instruction](/images/database/gdbInstruction.PNG)

Here we can see that the value located at ```0x14(%rax)``` is being compared to the literal hex string of ```0xBEEFCAFE```. With that in mind, we just need to try different ways to overwrite that memory address with our username value.

My first tests involved throwing a large number of 'A's at the input to see what would happen. 

![Bunch of As](/images/database/bunchOfA.PNG)

I noticed that after 19 characters I started getting a permission denied error. If we look back at the code snippet from earlier, this means we have changed the value of ```admin->role``` and the else branch is now being followed instead of the first if. This is great news and means we are one step closer to taking the else if branch and getting the flag.

## Exploit:
With all of this information gathered I was ready to try putting together an exploit. I used the tried-and-true python -c method for feeding in non-ascii characters into our binary.
My payload ended up being a padding of 20 A's, followed by 0xBEEFCAFE reversed to account for endiness
```python
python3 -c 'import sys;sys.stdout.buffer.write(b"A"*20+b"\xFE\xCA\xEF\xBE")'
```
I tested this on my offline copy of the challenge and got this:

![Offline Exploit](/images/database/offlineExploit.PNG)

Perfect! Exactly as we expected! Now it's time to switch over to the live version and cross our fingers.

![Final PWN](/images/database/solved.PNG)

Just like that we're in. Very fun challenge that used a less common vulnerability for some simple pwning

```
ractf{w0w_1_w0nD3r_wH4t_free(admin)_d0e5}
```
