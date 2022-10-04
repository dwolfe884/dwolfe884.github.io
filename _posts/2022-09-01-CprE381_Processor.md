---
title: CprE 381 Processor Design
date: 2022-09-01 8:00:00 +/-1111
author: David Wolfe
categories: [Projects, Iowa State University]
tags: [project,ISU,portfolio] 
---
![Processor Diagram](/images/381/hw.png)
## Intro
This project was the most time consuming and effort intensive projects I undertook during my time at Iowa State University. The deliverable for this project was a 5 stage pipelined proceessor with hardware enforced data forwarding capabilities. This required our team to use everything we had learned about CPU design and more in order to finish on time.

## My Role
I became the team lead for my group and was responsible for delegating out tasks and bringing the different parts of the processor together. I also wrote a number of the more complex, larger scale components such as the control unit (CU) and the arithmatic logic unit (ALU). In order to manage version control and component integration I created a github repo to host our project. You can find our final project [here](https://github.com/dwolfe884/381processor)

## Challenges Faced
My team and I faced a number of challenges that slowed our progress to a crawl. One very annyoing issue we ran into came in the form of a poorly coded Arithmetic Logic Unit. When making this specific part of the CPU our team didn't pay enough attention to the specific inputs and which input went to which operations. We focused too much on the operations in which math was reversable (ie: 1+4 == 4+1) and not enough on the ones in which order matters (ie: 1-4 != 4-1). After hours of carfully crafted test cases and lots of manual recording of register values we traced the issue always down to a set of specific AND gates inside of our ALU. We flipped one of the inputs for these AND gates and all of the sudden everything worked.

## Lessons Learned
CprE 381 will be the class I remember the most from my undergraduate degree program. It was the class that brough all of the different computing concepts I had leared up to this point together and made me realize what it takes to run a computer. Before taking 381 I understood that computers operated on 1's and 0's. I also knew how to program in languages like C. I was missing out on the knowledge of how computers translate that code into 1's and 0's and how those bits get carried around the physical hardware of a CPU to make anything happen. 381 forced me to not only experience these aspects of computer but to truly learn them.
