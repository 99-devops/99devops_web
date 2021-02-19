---
title: "Life is hard, randomness is harder !"
date: 2021-02-12T14:20:12+11:00
draft: false
description: "A descriptive guide about randomness in linux system and when to use /dev/random or /dev/urandom."
tags: [devops, linux]
categories: [linux]
---

![random](/img/random.png)

"You might think that creating random number is easy but its not as easy you think. Entropy as we know is the state of randomness, The more entropy we get, the more randomness there is."

# TL;DR

True Randomness is hard. Cannot be achieved.  Use /dev/urandom for your UNIX apps in day-to-day use. 


## First of all, what is Randomness, Randomness in what ?

According to Wikipedia "randomness is the apparent lack of pattern or predictability in events". When we say anything is random means that it has no order, or predictability. Individual random events are unpredictable. Anything is said random only where the outcome cannot be predicted in advance. Example: If you toss a coin 10 times, what is the possibility that the order of events will be H, H, T, T, H, T. You do not know the sequence; this unpredictable chain of events is known as random events. This is randomness. 

## Is Entropy truly random ?

Entropy is the degree of disorder in a system; for a computer system the "entropy pool" is a measure of the available sources of randomness. High available entropy means lots of sources of randomness, from human input devices (timing and movement events from mice, keyboards), from physical IO (spinning disk movements, network traffic) and/or from dedicated hardware (thermal noise diodes, CPU hardware). The pool "depth" varies with these processes and can be zero - for example a system lacking any sources of randomness (e.g., an IoT device that has very few sources of randomness, or a Docker container running in the cloud) will have a very shallow entropy pool. 


## How to get a truly random random number ?

In order to get a truly random number you need to have high entropy. Higher the entropy, higher the randomness and higher the value. So, how does one get higher entropy? The only events that can be said truly random are the natural phenomena or physical events. What's the chance you press share button and share this article? No one knows, it's random and cannot be predicted ;).

## Why its important to generate true random numbers ?

It is required for security, effective hashing algorithms (e.g., for databases and load balancing) requires good randomness.  Generally, there are different algorithms that use generation of random number which are in turn used to create crypto keys. If their numbers are not random then the cryptographic keys in turn will be weaker and could be predicted relatively easily. This is the major reason finding ways to generate true random numbers. 

## How to Linux generates random numbers ?

Machines or OS are dumb. They only do what they are programmed to do. Machine are given set of instructions which they execute to get a random number.  Computers are deterministic – that is, predictable. Lacking any source of entropy, machines instead use a pseudo-random number generator (PRNG) They simulate randomness using algorithms and those algorithms generates a sequence of numbers that looks random. Note that, it will generate random looking sequence of number but how to make them random? 

This is the reason nature or human are involved in generating true randomness into a system. Physical activities like typing, moving mouse, audio, video data tend to be random that we want and involving them into the system could give us more entropy which caused entropy to go high, and higher the entropy, higher the randomness. 

Here, PRNG (Pseudo Random Number Generator) comes into the picture. PRNG uses algorithms to generate random looking number. . Linux uses Pseudo Random Number Generator (PRNG) or Cryptographically Secure Random Number Generator (CSPRNG). These use mathematical formulas to achieve maximum randomness. PRNG is a set of mathematical formulas implemented in program to generate large quantities of random digits that are needed.  

You can see the image here on how PRNG works 

![prng](/img/prng.png)


It first starts with a seed value which is used as the initial value and PRNG uses that seed and using the algorithms it generates the random number, when you ask for next number then it has formula to generate the next number and the next and the next. 

Many programs do not have their own source of generating truly random bits hence they instead they use PRNGs to generate these numbers. Pseudo random because it’s not possible to generate true random number from deterministic things like computer. 

PRNG uses available entropy to generate random numbers. But how is entropy collected? You can see from the following image 

![prng](/img/prng-2.png)

As we discussed, true way to get higher entropy is from random operations like keyboard typing, mouse movements, audio data, video source. Once the entropy source is collected, it is kept in the entropy pool, and every time the PRNG requires a random seed it no deterministically chooses a seed from the pool. Once PRNG seed is applied, PRNG uses formulas to generate random data as mentioned above. They make random character data and make it available to OS and process to use through special files called /dev/urandom or /dev/random. Some Linux systems like fedora allows multiple entropy sources as well. /dev/random is the 'raw' RNG - it will block when no entropy is available - should not be used as it can cause your system to "hang" waiting for entropy. /dev/urandom is the non-blocking PRNG and is the one you should always use 

When you start your computer, it has low entropy and when you move around play with keyboards, open close programs the entropy is accumulated and PRNG uses these noise sources to generate entropy pool which PRNG uses to generate random numbers.  

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

let's see how you can use up all your available entropy. Run the above watch command on the terminal, and on another terminal, we will use that to generate a random bit number. When you run the command to generate random bit, that entropy accumulated will be used up and it will be zero again. 

let's run this. Currently my entropy available is (From above command) 
``` 
Every 1.0s: cat /proc/sys/kernel/random/entropy_avail                                                                              Thu Feb 18 04:22:31 2021 

989 
``` 

now, once i run a command to generate random bits, it is all used up and starts again to accumulate entropy. Run this command and watch entropy of the system go down. 
``` 
cat /dev/random > random_bits 
``` 

now my entropy is used up and it's starting again which i can check using following command 
``` 
# cat /proc/sys/kernel/random/entropy_avail 

2 
```

It's shown in this GIF file
![randomness](/img/random.gif)

## What are /dev/urandom , /dev/random , /dev/arandom ?

These are the PRNG that we were talking about. It is a special file. It collects noise (entropy) from different sources and generates a random number. The major different between all three of them if that /dev/random blocks process if there is not enough entropy available than requested whereas /dev/urandom never block even if PRNG has not fully initialized the seed when we boot the computer or restart it.  

Each PRNG has a generator which has estimated entropy pool size which we saw above. Random numbers are generated from this pool. When /dev/random is used, it checks if there are enough number of sizes in entropy pool and only return random bytes when there are estimated number of bits of noise in entropy pool. When its empty /dev/random blocks until additional environmental noise is gathered. This is used to serve for CSPRNG (Cryptographically Secure Pseudorandom Number Generator) which can be used to deliver better random number with highest entropy as possible.  

On the other hand, /dev/urandom is a non-blocking generator which re-uses the available entropy pool to generate more random bits.

## Which one to use /dev/urandom or /dev/random ?

Generally, /dev/random and /dev/urandom are used as /dev/arandom is like /dev/urandom/. /dev/urandom should always be used unless you have a specific need for “cryptograhic strength” randmoness. /dev/random is used when there is security in the picture and randomness needs to be secure against malicious actors. When system boots, the system is less on entropy and random bits generated from /dev/urandom could not be truly random. 

It is recommended to use /dev/urandom which your program is generatng random numbers as we do not want application to halt because system does not have enough entropy. It just needs a random number which can be given by /dev/urandom.

If the application is mission critical and is very closely dependent of security, then use /dev/random as we can get more random data using this compared to /dev/urandom but still random bytes provided by /dev/urandom produces good random bytes of cryptographic quality which it gets from OS during startup.

For almost all programs use /dev/urandom because of its non-blocking state.


#### OpenSSL generating random data

OpenSSL uses its own pseudo random number generator (PRNG), seeded on startup from a source of random data provided by the operating system. It uses <code>rand.c</code> which uses /dev/urandom which can be seen from the system call. 

Here is openssl generating random bytes of data, where you can see it also uses /dev/urandom 
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

In python random number are generated using <code>random.seed(a=None, version=2)</code> in which a is the seed. If value of 'a' is not defined the OS system time is used, and if entropy seed as we discussed above is provided then it is used and With version 2 (the default), a str, bytes, or byte array object gets converted to an int and all of its bits are used. 

Here is a simple python script that generates random floating-point number. If you see the system calls that are used to run this program and get random number, it also uses /dev/urandom 
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

## References

https://wiki.archlinux.org/index.php/Random_number_generation
https://wiki.openssl.org/index.php/Random_Numbers
https://eprint.iacr.org/2012/251.pdf
https://en.wikipedia.org/wiki//dev/random
https://openjdk.java.net/jeps/356
https://en.wikipedia.org/wiki/Cryptographically_secure_pseudorandom_number_generator
https://linux.die.net/man/3/open
https://unix.stackexchange.com/questions/324209/when-to-use-dev-random-vs-dev-urandom
https://en.wikipedia.org/wiki/Yarrow_algorithm