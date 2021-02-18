---
title: "Life is hard, randomness is harder?"
date: 2021-02-12T14:20:12+11:00
draft: false
description: "A descriptive guide when to use /dev/random or /dev/urandom or /dev/arandom in linux"
tags: [devops, linux]
categories: [linux]
---

![random](/img/random.png)

"You might think that creating random number is easy but its not as easy you think. Entropy as we know is the state of randomness, The more entropy we get, the more randomness there is."

# Table of Contents

[First of all, what is Entropy ?](#-First-of-all,-what-is-Entropy-?)

[Is Entropy truly random ?](#Is-Entropy-truly-random-?)

[How to get high entropy ?](#How-to-get-high-entropy-?)

[Why its important to generate true random numbers ](#Why-its-important-to-generate-true-random-numbers-?)

[How to Linux generates random numbers ?](#How-to-Linux-generates-random-numbers-?)

[What is the entropy of my system ?](#What-is-the-entropy-of-my-system-?)

[What are /dev/urandom , /dev/random , /dev/arandom ?](#What-are-/dev/urandom-,-/dev/random-,-/dev/arandom-?)

[Which one to use /dev/urandom or /dev/random ?](#Which-one-to-use-/dev/urandom-r-/dev/random-?)


# TL;DR

Randomness is hard. High entropy is needed for better randomness. True randomness cannot be achieved, in fact true randomness means entropy being the highest which is impossible to achieve but we can include events outside systems like physical events to make randomness more random.  Use /dev/urandom for your application for day to day use.

## First of all, what is Entropy ?

Entryopy is the measure of randomness or disorderness in a system where as Randomness is a phenomenon or procedure for getting data that is random. It is a measure of randomness and it increases everytime the randomness in system increases. Now another question, what is randomness ? Randomness in what ?

## Now what is Randomness then, Randomness in what ?

According to wikipedia "randomness is the apparent lack of pattern or predictability in events". When we say anything is random means that it has not order, sequence. Individual random events are by definition unpredictable. Anything is said random only where The outcome cannot be predicted in advance. Example: If you toss a coin 10 times, what is the possibility that the order of events will be H, H, T, T, H, T. You do not know the sequence, this unpredictable chain of events is known as random events. This is randomness.

## Is Entropy truly random ?

As discussed above, entropy is the measure of the randomness of a system. Meaning, is the system has low entropy then that means that there are a lot of repetitions in the events. Repetitions invites patterns and if you start getting pattern then you can guess the outcome and now its not random anymore. So lower entropy has less randomness which mean its doesnot tend to be as random as it wants to be. 


## How to get a truly random random number ?

In order to get a truly random number you need to have high entropy. Higher the entropy, higher the randomness and higher the value. So, how does one get higher entropy ? The only events that can be said truly random are the natural phenomena or physical events. What's the chance you press share button and share this article ? No one knows, It's random and cannot be predicted ;).

## Why its important to generate true random numbers ?

It is due to security reasons.  Generally there are different algorithms that use generation of random number which are inturn used to create crypto keys. If there numbers are not random then the cryptographic keys inturn will be weaker and could be predicted relatively easily. This is the the major reason finding ways to generate true random numbers.

## How to Linux generates random numbers ?

Machines or OS are dumb. They only do what they are programmed to do. Machine are given set of instructions which they execute to get a random number. If there is no human event, eventually they will start creating pattern for that randomness. There is not way computer can perform those random events, so computer use so called Pseudo random numbers. They simulate randomness using algorithms and those algorithms generates a sequence of numbers that looks random. Note that, it will generate random looking sequence of number but how to make them actually random ?

This is the reason nature or human are involved in generating true randomness into a system. Physical activities like typing, moving mouse, audio, video data tend to be random that we want and involving them into the system could give us more entropy which caused entropy to go high, and higher the entropy, higher the randomness.

Here, PRNG (Pseudo Random Number Generator) comes into the picture. PRNG uses alrogithms to generate random looking number. Linux was the first OS to implement PRNG into its kernel. Linux uses Pseudo Random Number Generator (PRNG) or Random Number Generator (CSPRNG). These use mathematical formulas to achieve maximum randomness. Linux was the first OS to include PRNG on its kernel. PRNG is a set of mathematical formulas implemented in program to generate large quantities of random digits that are needed. 

You can see the image here on how PRNG works

![prng](/img/prng.png)


It first starts with a seed value which is used as the initial value and PRNG uses that seed and using the alrogithms it generates the random number, when you ask for next number then it has formula to generate the next number  and the next and the next.

Many programs do not have their own source of generating truly random bits hence they instead they use PRNGs to generate these numbers. Pseudo random because its not possible to generate true random number from deterministic thing like computer.

PRNG uses available entropy to generate random numbers. But how is entropy collected ? You can see from the following image

![prng](/img/prng-2.png)

As we discussed, true way to get higher entropy is from random operations like keyboard typings, mouse movements, audio data, video source. Once the entropy source is collected, it is kept in the entropy pool, and everytime the PRNG requires a random seed it no deterministically chooses a seed from the pool. Once PRNG seed is applied, PRNG uses formulas to generate random data as mentioned above. They make random character data and make it available to OS and process to use through special fieles called /dev/urandom or /dev/random or /dev/arandom. Some linux systems like fedora allows multiple entropy sources as well. 

When you start your computer, it has low entropy and when you move around play with keyboards, open close programs the entropy is accumulated and PRNG uses these entropy to generate random numbers.


## What is the entropy of my system ?

You can check the entropy available in system using following command
```
# cat /proc/sys/kernel/random/entropy_avail

1047
```

You can see the entropy accumulation in your system using following command. Run following command in a terminal and look at the value increasing when you do anything else. 

```
watch -n 1 cat /proc/sys/kernel/random/entropy_avail
```

let's see how you acn use up all of your available entropy. Run the above watch command on the terminal, and on the another terminal we will use that to generate a random bit number. When you run the command to generate random bit, that entropy accuumulated will be used up and it will be zero again.

let's run this. Currently my entropy available is (From above command)
```
Every 1.0s: cat /proc/sys/kernel/random/entropy_avail                                                                              Thu Feb 18 04:22:31 2021

989
```

now, once i run  a command to generate random bits, it is all used up and starts again to accumulate entropy. Run this command and watch entropy of the system go down.
```
cat /dev/random > random_bits.txt
```

now my entropy is used up and its starting again which i can check using following command
```
# cat /proc/sys/kernel/random/entropy_avail

2
```

It's shown in this GIF file
![randomness](/img/random.gif)

### What are /dev/urandom , /dev/random , /dev/arandom ?

These are the PRNG that we were talking about. It is a special file. It collects noise (entropy) from different sources and generates a random number. The major different between all three of them if that /dev/random blocks process if there is not enough entropy available than requested where as /dev/urandom never block even if PRNG has not fully initialized the seed when we boot the computer or restart it. /dev/arandom blocks after boot until seed has been securely intialized through enough entropy and then never blocks again. Only few OS procive /dev/arandom. 

Each PRNG has a generator which has estimated entropy pool size which we saw above. Random numbers are generated from this pool. When /dev/random is used, it check if there are enough number of size in entropy pool and only return random bytes when there are estimated number of bits of noise in entropy pool. When its empty /dev/rabom blocks until additional environmenal noise is gathered. This is used to serve for CSPRNG (Cryptographically Secure Pseudoarndom Number Generator) which can be used to deliver better random number with highest entropy as possible. 

on the other hand /dev/urandom is a non blocking generator which re-uses the available entropy pool to generate more random bits. When you read from /dev/urandom it will 

If your system does not have /dev/random or /dev/urandom, it can be generated using following command
```
           mknod -m 666 /dev/random c 1 8
           mknod -m 666 /dev/urandom c 1 9
           chown root:root /dev/random /dev/urandom
```

## Which one to use /dev/urandom or /dev/random ?

Generally /dev/random and /dev/urandom are used as /dev/arandom is similar to /dev/urandom/. /dev/urandom is used in plce where there is constant need of random numbers and the actual randomness is not too much important while /dev/random is used when there is security in the picture and randomness needs to be reliable. When system boots, the system is less on entropy and random bits generated from /dev/urandom could not be truly random.

It is recommended to use /dev/urandom which your program is generatng random numbers as we do not want application to halt because system does not have enough entropy. It just needs a random number which can be given by /dev/urandom.

If the application is mission critical and is very closely dependent of security, then use /dev/random as we can get more random data using this compared to /dev/urandom but stil random bytes provided by /dev/urandom produces good random bytes of cryptographic quality which it gets from OS during startup.

For almost all programs use /dev/urandom because of its non-blocking state.

#### OpenSSL generating random data

OpenSSL uses its own pseudo random number generator (PRNG), seeded on startup from a source of random data provided by the operating system. It uses <code>rand.c</code>. from the system call Here is openssl generating random bytes of data, it also uses /dev/urandom
```
strace -xe trace=file,read,write,close openssl rand 10
```

System calls:
```
..
..
..
open("/dev/urandom", O_RDONLY|O_NOCTTY|O_NONBLOCK) = 3
read(3, "\x1a\x54\x44\xa3\x76\x3b\x2c\xd3\x51\xd7\x83\x82\x2a\x27\x72\x45\x21\xbb\xa0\x32\xa3\x29\xe6\x6e\x22\xfb\xde\x5d\x56\xa0\x68\xd8"..., 48) = 48
close(3)
..
..
```


#### Python script generating random number

In python random number are generated using <code>random.seed(a=None, version=2)</code> in which a is the seed. If value of 'a' is not defined the OS system time is used, and if entropy seed as we discussed above is provided then it is used and With version 2 (the default), a str, bytes, or bytearray object gets converted to an int and all of its bits are used. 

Here is a simple python script that generates random number.If you see the system calls that are used to run this program and get random number, it also uses /dev/urandom
```
strace -xe trace=file,read,write,close python -c "import random; n = random.random(); print(n);"
```

System calls:
```
...
...
open("/dev/urandom", O_RDONLY)          = 4
read(4, "\x09\x98\x02\x02\x51\x93\x67\x14\xd1\x57\x96\xb6\x0d\xde\x10\xf8", 16) = 16
close(3)
...
...
```

FYI, 
O_RDONLY, O_NOCTTY, O_NONBLOCK are flags on file management system calls on Linux where O_RDONLY means open file for read only, O_NOCTTY means if its set and path identifies a terminal device, open() will not cause terminal device to become the controlling terminal for the process and O_NONBLOCK flag the open() function will return without blocking for the device to be ready or available. Subsequent behaviour of the device is device-specific.
