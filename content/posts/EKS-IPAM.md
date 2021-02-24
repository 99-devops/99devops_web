---
title: "Envoy proxy running but not ready - AWS EKS Istio"
date: 2021-02-24T16:31:31+11:00
draft: false
description: "Envoy proxy is NOT ready: config not received from Pilot"
tags: [linux, sre]
categories: [sre]
---

![aws_ip](/img/aws_ip.png)

I am sure, it's not just me who encountered this error while using EKS and istio. So, here is how i resolved the issue.

## Problem
```
Istio : Envoy proxy is NOT ready: config not received from Pilot.
```
or
```
failed to set up sandbox container "c54935a5c5982d95ef9391fdfff08b86bbf6b8c4b6568b4ee7bb81fce8a0f424" network for pod  ... network: add cmd: failed to assign an IP address to container
```


## Solution

The istiod on main cluster is exposed by NodePort service. This issue might be due ot 2 things:
1. AWS VPC IP ran out
2. CNI and Dockerd not working properly. 

In order to resolve first one, just check on aws console and check the number if ip address remaining.

For second, one list all the pod and find the aws node daemonset with runs the CNI and IPAMd. For EKS it is generally named, 'aws-node'. 
```
kubectl get pods -n kube-system                                                    
NAME                       READY   STATUS    RESTARTS   AGE
aws-node-lcxhc             1/1     Running   0          46m
aws-node-msk2t             1/1     Running   0          46m
coredns-559b5db75d-77wcq   1/1     Running   1          7d
coredns-559b5db75d-rm47q   1/1     Running   1          7d
kube-proxy-998jb           1/1     Running   1          6d23h
kube-proxy-z9rzv           1/1     Running   1          6d23  
```

Delete that pod and it will re-spawn the daemonset and will reinitialize CNI and IPAM daemon.


After reinitialization
```
 {"level":"info","ts":"2021-02-24T01:15:42.897Z","caller":"entrypoint.sh","msg":"Install CNI binary.."}                                                  │
│ {"level":"info","ts":"2021-02-24T01:15:42.915Z","caller":"entrypoint.sh","msg":"Starting IPAM daemon in the background ... "}                           │
│ {"level":"info","ts":"2021-02-24T01:15:42.916Z","caller":"entrypoint.sh","msg":"Checking for IPAM connectivity ... "}                                   │
│ {"level":"info","ts":"2021-02-24T01:15:44.935Z","caller":"entrypoint.sh","msg":"Copying config file ... "}                                              │
│ {"level":"info","ts":"2021-02-24T01:15:44.937Z","caller":"entrypoint.sh","msg":"Successfully copied CNI plugin binary and config file."}                │
│ {"level":"info","ts":"2021-02-24T01:15:44.938Z","caller":"entrypoint.sh","msg":"Foregrounding IPAM daemon ..."}
```

Now, deploy your app again, it should work.