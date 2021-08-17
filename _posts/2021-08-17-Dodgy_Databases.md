---
title: Dodgy Databases
date: 2021-08-17 12:00:00 +/-1111
author: David Wolfe
categories: [Writeups, RACTF]
tags: [ctf,writeup,ractf] 
---
## Description:
One of our most senior engineers wrote this database code, it's super well commented code, but it does seem like they have a bit of a god complex. See if you can help them out.

## Methodology

This challenge was under the pwn/reversing category and it came with source code. I first looked through the provided .c file for any obvious issues. The first noteworthy line of code I could find was on line 204.
![Free Vulnerability](/images/database/freeVuln.PNG)
In this snippet we can see the memory holding the admin object being freed and then immediantly creating a new user object after that. This sequence of events casues a ```Use-After-Free``` vulnerability since the now freed admin object is still being passed into the ```users_register_user``` function. On top of that, because these objects are the same size and structure, it's very likley that our new user object would reuse some/all of the memory originally allocated for the admin object.

I continued to read through this source code and found our goal, in this case where the flag is printed to the screen.
![Goal](/images/database/goal.PNG)
This function shows a simple check for if the passed in admin object has a role of ```ROLE_GOD```. If it does, we get the flag.

With that static analysis out of the way, I moved on to a more dynamic approach. I launched the compiled binary with GDB and set a breakpoint at the ```users_register_user``` function. After entering a random username we hit the breakpoint and are able to see the exact instruction where this comparison for ```ROLE_GOD``` is taking place.
![GBD Instruction](/images/database/gdbInstruction.PNG)
