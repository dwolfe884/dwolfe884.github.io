---
title: x86 Flow Graph Plugin for Obsidian 
date: 2023-04-14 09:00:00 +/-666
author: David Wolfe
categories: [Tools, Obsidian]
toolinfo: A plugin for my Obsidian vault to make assembly notes more useful
toolurl: https://github.com/dwolfe884/obsidian-x86-flow-graph
tags: [x86, Reverse Engineering, Obsidian, plugin, tools] 
---
## Overview:
Another quick post about a tool I've been developing. There I was, sitting in my reverse engineering class, writing down x86 assembly, trying to get used to reading common loops and other control flow patterns in raw assembly. This was around the same time that I noticed the new Canvas feature in Obsidian. Then it hit me, what of instead of studying for my assembly quizzes I wrote a plugin for Obsidian to automatically translate all my assembly notes into beautiful flow graphs similar to what Ghidra and Ida do! This project not only forced me to get more familiar with x86 control flow but it had the added bonus of teaching me more about how my favorite note taking app works. While this probably wont be very useful for anyone reading this, it did help me through a semester of note taking, so I think it's worth sharing. Right now this plugin has been designed to work specifically with x86 asm. However, it wouldn't be hard to adapt it to other ISA's.

## Uses and Demos:
This tool has one very specific use case.

1. Easier documentation of reverse engineering notes.

Below I've included a side by side of the assembly and the graph produced by my plugin. It's important to note that the graph will not be well laid out when it's first generated. Moving the nodes around in an order that makes sense is left to the user.

![Graph Demo](/images/obsidian/demo.png)

## Link:
[https://github.com/dwolfe884/obsidian-x86-flow-graph](https://github.com/dwolfe884/obsidian-x86-flow-graph)