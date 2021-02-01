---
title: "Are You Backing Up Your k8s Cluster? Did you tried restoring it ?"
date: 2021-01-25T14:20:12+11:00
draft: false
description: "Kubernetes cluster backup and restoring"
tags: [devops, k8s]
categories: [devops]
---
"Backup not tested is backup not done"

![Velero](/img/velero.png)

You might have heard this phrase so many time but it has the same impact even if you listen to it thousands more. If you are taking backup and not testing restoring it, then you are in a false dilemma and depending on sheer dumb luck. On this note,

For folks using k8s in production, here is a good tool that you can use.

https://velero.io/

But having said that, there are two sides to this story. There is another saying

"Best way to backup is not to backup at all".

Ideally you should set your systems to be provisioned, orchestrated and running from ground up with a click of a button.

If your systems are not the later, then it's better spend some time backup up and test restoring them.