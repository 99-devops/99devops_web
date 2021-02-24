---
title: "Preserving source IP address in L4 loadbalancer (AWS, DO) using Istio's Envoy Filter and Proxy protocol."
date: 2021-02-24T08:31:31+11:00
draft: false
description: "This article shows you how you can preserve source IP address in kubernetes using Proxy protocol and istio's envoy proxy filters."
tags: [linux, sre]
categories: [sre]
---

![Unmanaged](/img/envoy.png)

When you are using Istio, the source ip address gets replaces when it passes through the Loadbalancer and istio ingress, the source ip address gets changed.

## Problem

Source IP not being preserved, instead changed to either LB or Node IP address in L4 loadbalancer.  This causes issuse like cannot whitelist IP addres for ip filtering, cannot implement rate limiting, logging and visualizing will not work and access control using RBAC cannot be done.


### What is Proxy Protocol

It is a convenient way to send connection information like client IP address across multiple servers like NAT or proxies. It requires little change to infrastructure to limit performance impact. The PROXY protocol's goal is to fill the server's internal structures with the information collected by the proxy that the server would have been able to get by itself if the client was connecting directly to the server instead of via a proxy. The information carried by the protocol are the ones the server would get using getsockname() and getpeername().

If you check the istio-ingress log then you would only see the source add as the Node private address if "externalTrafficPolicy" was set to "Cluster" and Node public address if set to "Local". Which would look like this:
```
kube
│ [2021-02-24T06:30:40.015Z] "GET /spec.json HTTP/2" 200 - "-" 0 41019 6 5 "10.234.<REDACTED>" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.192 Safari/537.36" "7c28536b-b6d4-9442-ab6c-744715ed82bf" "httpbin.<DOMAIN>│
```

As you can see the source ip is shown as 10.234 range which is the node ip rather than actual source IP. This invites a lot of issues for security, monitoring and ACLs.

## Solution 

In order to preserve source IP, you would need to enable proxy protocol in the underlying loadbalancer whether it be DO or AWS classic LB.

First,
1. Annotate / Enable proxy protocol on the Load balancer.
2. Deploy Envoy filter to look for proxy protocol headers.

### Using DO Loadbalancer


For DO LB you can run following command to annotate and enable proxy protocol

```
kubectl annotate --overwrite svc istio-ingressgateway service.beta.kubernetes.io/do-loadbalancer-enable-proxy-protocol="true" -n istio-system  
```

### Using AWS Loadbalancer

If you are using classic loadbalancer for EKS.

```
kubectl annotate --overwrite svc istio-ingressgateway service.beta.kubernetes.io/aws-load-balancer-proxy-protocol="*" -n istio-system  
```

## Use Envoy filter to parse proxy protocol using HTTP listeners
Apply following filter which will look for headers information from proxy protocols.

```
cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: proxy-protocol
  namespace: istio-system
spec:
  workloadSelector:
    labels:
      istio: ingressgateway
  configPatches:
  - applyTo: LISTENER
    patch:
      operation: MERGE
      value:
        listener_filters:
        - name: envoy.listener.proxy_protocol
        - name: envoy.listener.tls_inspector
EOF
```

Now if you check your logs from istio-ingress pod, it should now start showing the source IP address.
```
kube
│ [2021-02-24T06:30:40.015Z] "GET /spec.json HTTP/2" 200 - "-" 0 41019 6 5 "60.<REDACTED>" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.192 Safari/537.36" "7c28536b-b6d4-9442-ab6c-744715ed82bf" "httpbin.<DOMAIN>│
```

now you can see its getting 60.XX range as source ip which is the actual source IP. Now you can use Istio authorization policy for RBAC and ACls and do a lot of things with it.