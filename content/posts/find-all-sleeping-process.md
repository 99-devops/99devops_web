---
title: "Find All Sleeping Process"
date: 2020-12-02T21:13:15+11:00
draft: false
tags: [linux, sre, interview]
categories: [linux, interview]
---
Hello today, i am going to show you how you can find all the processes that are sleeping.

![Unmanaged](/img/sleeping.png)

First you need to see what are the fields in ps command.

Execute
```
ps -aux | less
```

this shows you all the fields on top which you can use later.

Now using awk and pipes, we grab all the process with state “S” for sleeping or “D” for processes in uninterruptible sleep.

Command:
```
ps -eo state,pid,command | awk ‘{if($1==”S” || $1==”D”) print($1,$2,$3)}’
```


ps -eo state,pid,command shows every process (-e) with output formatted with (-o) to get desired output format.


So, this command outputs every process in mentioned format.

|| -> It is the pipes in linux

awk is used to manipulate data. if else condition is same as other language.

$1 is the first filed in output, $2 is the second and $3 is the third.

Command:
```
awk ‘{if($1==”S” || $1==”D”) print($1,$2,$3)}’
```

This command manipulates output with our condition to see if it has state S or D and print it.