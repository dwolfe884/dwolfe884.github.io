---
title: Senior Design
date: 2022-09-02 8:00:00 +/-1111
author: David Wolfe
categories: [Projects, Iowa State University]
tags: [project,ISU,portfolio] 
---
![Closed Loop System Diagram](/images/sd/sysloop.png)

## Description of project 
The senior design project I was assigned to work on was `CySecAgri`. The deliverable for this project was a user friendly web application that aggregated farming IoT sensor data, processed it into user friendly graphs and charts, and finally ran intrusion detection algorithims against it to detect if there was a threat actor or other issue facing the sensors or the crops they were monitoring.

## Your role 
Being one of the 3 cyber security engineering majors on this team I was tasked with ensuring our senor data was encrypted and trasmitted securly between our cloud storage and the finally users browser session. This data contained a number of different measurements that could be considered sensitive depending on the specific implementation of our application.

## Skills or knowledge gained 
In order to properly protect our data in transit I learned about many different attacks that could be possible against our applications data. While researching additional attack vectors for this project I found that physical security was going to need to be a more important aspect than I previously considered. Normally in my cyber security work physical security is almost never considered because the machines we are working on are deep inside of company datacenters. However, this project requires us to stick a fully functioning computer in the middle of a field with no added security around it. I plan on implementing several mitigations for common physical attacks such as disabling the USB ports and locking the storage container the computer is help in. I also learned new techniques and common practices for ensuring data is stored securly in the cloud as I was responsible for the API key our basestation used for posting data to AWS.

## Link supporting documents 
In the first set of meetings with our client we decided it was important to create a closed loop system diagram to get a better idea of the exact scope of our project. This diagram and a number of other documents that ensured we acomplishe dour goals can be found on our shared [google drive](https://drive.google.com/drive/folders/1zYstwKIJDPYnWsC2eBpmDFC3hkMTf6Nm?usp=sharing). I was specifically responsible for the System Loop diagram and parts of the Use Case diagram.

## Big picture contribution
While we haven't been able to start development of our final product yet do to slow replys on behave of the current owners of the IoT hardware we plan on using. I will have a working prototype for our cloud data transmitter by the end of the Fall 22 semester. This code will most likely be written in Python and run on a Raspberry Pi. While researching the best way to interact with the cloud I've found that I will most likely need to make this program multithreaded to handle the massive amount of data we will be moving in and out of the cloud on a daily basis.
