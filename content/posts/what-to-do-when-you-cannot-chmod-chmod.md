---
title: "What to Do When You Cannot Chmod Chmod"
date: 2021-01-28T21:21:33+11:00
draft: false
tags: [linux, interview, sre]
categories: [interview, linux]
---

When you cannot chmod chmod? what to do? This might help to re-instate permission back.

Here are following techniques where you can use chmod to change permission when chmod itself is not executable.

Using rsync:

The first technique is to use rsync where you copy the chmod binary into another location with adding permission to  file being copied.

![Unmanaged](/img/first.png)


Using setfacl:

Another way is one of my favorites where you first use setfacl to set executable permission to chmod for the owner, then use the owner permission to set permission to groups and others.



![Unmanaged](/img/second.png)


Using dynamic linker:

Third one is the easiest of them all. Using dynamic linker.

![Unmanaged](/img/third.png)


Using perl:

Since most of the linux system are pre built with perl, we can use perl to execute command which can be used to change permission.


![Unmanaged](/img/fourth.png)


Using busybox:

Another method is using busybox. It is a software which provides various linux/unix utilities in a file.
![Unmanaged](/img/fifth.png)


Using dd and chown

We can change permission for chmod using dd and chown which is used to convert and copy files.

![Unmanaged](/img/sixth.png)


Using python

If you are very good in python then you can set it up using python script with os module and execute it from there to change permission.



![Unmanaged](/img/seventh.png)

Burrowing permission from other executable

This this step we are basically borrowing permission from another executable file.

![Unmanaged](/img/eight.png)


Using install

Using install we can use to copy file from source to destination with modified permission.

![Unmanaged](/img/nine.png)


Also. some of the commands are executed with sudo privilege. If you don't have it then make sure you first run some privilege escalation techniques. One of which is to use an exploit dirtycow.

Link: https://github.com/dirtycow/dirtycow.github.io/wiki/PoC

