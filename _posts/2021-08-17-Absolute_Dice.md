---
title: Absolute Dice
date: 2021-08-17 10:00:00 +/-1111
author: David Wolfe
categories: [Writeups, RACTF]
tags: [ctf,writeup,ractf] 
---
## Description:
```
Man, the final boss of this game is kickin' my ass! Can you give me a hand?
```
We are given only the challenge executable and an IP:Port for the real exploit.
## Methodology
Since we don't have any source code to look at for this challenge. I started by simply running the offline program and seeing what happens.

![Normal Behavior](/images/dice/goal.PNG)

It's always good to take stock of what normal operation looks like before diving into the technical parts of a challenge. With these tests run I tossed the binary in Ghidra to decompile and start looking at the code a little closer.

![Raw Decompile](/images/dice/rawCode.PNG)

Thankfully the binary is not stripped and Ghidra is able to present pretty much the entire program to us. Some important things to note in this screenshot are:
1. We only get 100 guesses before the program exits
2. The program is using C's built in rand function
3. The rand function is being seeded with srand at the start of every loop. 

If we are to be tasked with guessing a totally random number 30 times in a row. It's probably best to look at srand and rand first. I launched the binary with GDB and set a break point just before the calls to ```fopen()```, ```fread()```, and s```rand()```.

![Reading /dev/urandom](/images/dice/devrand.PNG)

As you can see in this screenshot, we are able to see that the file being opened is linux's ```/dev/urandom```. From this file 4 bytes of data are read in and used as the seed for srand.

This is very interesting. I can't see any problem with using /dev/urandom to generate random noise to use as the seed for rand(). I was stuck here for a while. I read a lot of blog posts about predicting random numbers and breaking different systems. However, all of those posts assumed that the seeding of the random number generator was flawed in some way and this challenge didn't seem to have any issues.

I ended up diving back into ghidra to see if I had missed anything in the code. That's when I found 1 line of code that made 0 sense to me.

![Out of Bounds](/images/dice/overflow.PNG)

In this screenshot guess is initialized as ```int guess[4]``` and holds:
* guess[0] = number of attempts
* guess[1] = the boss' guess
* guess[2] = number of correct guesses in a row
* guess[3] = player guess

So the question is, why would there be a line that takes the number of guesses, mods by 0x21 and adds 5? And then tries to index the guess array by that number?? At a minimum this line would produce ```guess[6]``` which would still be out of bounds. This line confused me, I thought it might of been a bug with the decompiler or something. Again, this is a mystery that went unsolved for somei time.

After staring at that line for far too long, I decided to write a python script to automatically guess numbers in an attempt to find some repeating pattern. I didn't find any patterns, but I did notice something different...

![Number of Guesses](/images/dice/32.PNG)

Rememeber all the way back at the beginning of this post when I said the program gives us 100 guesses before exiting? That's not what I was seeing with my script. I was consistently given 32 guesses before the socket just closed. Nothing in the code indicated that it would disconnect or exit after 32 guesses so I knew something important was happening. I ran my script against my local version of the program and found the root cause.

![Seg Fault](/images/dice/segfault.PNG)

We were getting a segmentation fault after 32 guesses! This made so much sense after what I had just learned about from that index out of bounds line in Ghidra. The max number from the line ```guess[0] % 0x21 + 5``` is 37 and it occurs when the number of guesses (guess[0]) equals 32.

Knowing this I started running gdb and giving it 31 random guesses and then stopping for deeper analysis on guess 32 to see why exactly this fault was occuring. From the above screenshot we are able to see that the fault is occuring inside of the ```fread()``` function. So, on guess 32 I gave it the string ```123456``` and jumped to the code where it tries to open ```/dev/urandom``` and read in data.

![Replaced Data](/images/dice/myinput.PNG)

Aha! The program is segfaulting because the 32nd guess gets copied perfectly into the spot in memory that normally points to the string for ```/dev/urandom``` This immediently gave me an idea for an exploit path and I started gathering data and developing my payload.
## Exploit:

We are now able to control the file that is used to seed the random number generator after 31 incorrect guesses. All we have to do now is replace ```/dev/urandom``` with a file that has constent data and we should be able to get consistant, predictable numbers. I searched the original binary for strings to try and find any paths to different files in memory. 
![New File String](/images/dice/fileString.PNG)

Confienently there was one such string. The string ```/lib/ld-linux.so.2``` was being stored at ```0x08048154``` in memory. To craft my payload I took that hex value and converted it to decimal ```134512980``` and placed it as my 32nd guess after 31 0's. I then tested this in GDB

![Changed String](/images/dice/changedString.PNG)

We were able to successfully change the file that is being opened!

![Pattern](/images/dice/pattern.PNG)

After giving it that new file we can see the game only guesses the number 4! This makes perfect sense since our new file has constant data and ```srand()``` is being seeded the exact same every loop.

Finally, it was time to put it all together into a python script and see what happens. My final python payload was:
```python
payload = ['0']*31 + ['134512980'] + ['4']*35
```
![Get Pwned](/images/dice/finalAnswer.PNG)

There it is! This was such an exciting last minute solve for me and my team. We started with a bunch of lose ends and questions but were eventually able to tie it all together for the flag. This might be one of my favorite challenges from this CTF.

```
ractf{Abs0lute_C0pe--Ju5t_T00_g00d_4t_th1S_g4me!}
```
