---
title: "Remote access authentication and authorisation using self signed certificates"
date: 2021-02-20T08:31:31+11:00
draft: false
description: "Article describing how we do remote access authentication and authorisation using self signed certificates"
tags: [devops, sre, k8s]
categories: [sre, k8s]
---

![random](/img/k8s_sec.png)

# Introduction

As we know that using kubeconfig for day to day use is a bad practice, instead each user should be given access based on their account and his/her request should be protected using self signed SSL. This way admin can enforce RBAC on the account and also revoke account if needed.

Here is how you can create user account on k8s and authenticate them using certificates.

# Pre-requisites:
* Administrator access to the production do cluster
* OpenSSL installed

# Guidelines for providing access to the cluster.

## Creating Certificate Signing Requests for New Users

We will be using a certificate-based authorization mechanism for the cluster. So, for this, you are required to generate an X509 certificate. This is to secure local authentication using TLS/SSL certificates.

Steps to follow

* Generate RSA key using the following command

```
export USER=<FIRSTNAME-LASTNAME>
export GROUP=developers
mkdir -p ~/certs
openssl genrsa -out ~/certs/$USER.key 4096
```

* Run the following command to create a CSR configuration file from the following template.
```
cat <<EOF > $USER.csr.cnf
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
[ dn ]
CN = $USER              
O = $GROUP
[ v3_ext ]                                
authorityKeyIdentifier=keyid,issuer:always
basicConstraints=CA:FALSE                
keyUsage=keyEncipherment,dataEncipherment
extendedKeyUsage=clientAuth
EOF
```


* Create a CSR (certificate signing request) 
```
openssl req -config ~/certs/$USER.csr.cnf -new -key ~/certs/$USER.key -nodes -out ~/certs/$USER.csr
```

* Check and verify user and group in the certificate signing request using the following command. 
```
openssl req -in ~/certs/$USER.csr -noout -text
```

## Managing Certificate Signing Requests with the Kubernetes API
TLS certificates issued to k8s API can be approved or denied using kubectl command. This gives the ability to ensure that the requested access is appropriate for the given user. Check for the group, as that group will play a major role. 

* Get base64 encoding of the CSR 
```
export BASE64_CSR=$(cat ~/certs/$USER.csr | base64 | tr -d '\n')
```


* Send CSR to DOKS cluster 
```
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: $USER
spec:
  groups:
  - system:authenticated
  request: $BASE64_CSR
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```

* Outout:
```
certificatesigningrequest.certificates.k8s.io/prabesh-thapa-authentication created
```

* Check for the CSR 
```
kubectl get csr

NAME                        AGE   SIGNERNAME                            REQUESTOR               CONDITION
prabesh-thapa-authentication   5s    kubernetes.io/kube-apiserver-client   prabesh.thapa@99devops.com   Pending
```

* Now, you need to approve the CSR request. 

```
kubectl certificate approve prabesh-thapa-authentication
```

* Check for CSR status again

if it approved and a certificate has been issued it will show something like this 

```
kubectl get csr                       

NAME         AGE   SIGNERNAME                            REQUESTOR               CONDITION
pradip-silwal   25s   kubernetes.io/kube-apiserver-client   prabesh.thapa@99devops.com   Approved,Issued
```

* If it approved and certificate issuing has failed it will show something like this 

```
kubectl get csr                                      
NAME                        AGE     SIGNERNAME                            REQUESTOR               CONDITION
prabesh-thapa-authentication   3m54s   kubernetes.io/kube-apiserver-client   prabesh.thapa@99devops.com   Approved,Failed
```

* Once CSR is approved, download the certificate using following command 

```
kubectl get csr $USER -o jsonpath='{.status.certificate}' | base64 --decode > ~/certs/$USER.crt
```

## Preparing kubeconfig for the user
Create a copy of your kubeconfig file and remove everything except the top 17 lines and add users data. Create a kubeconfig file for user and send it to them along with the key and crt file.

* make sure the certificate and key file location are properly defined.


Example:
```
cat <<EOF > config-$USER
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLLSFTIPSMDIJERMLKSDKJKJLDFKJLRS0tLS0tCk1JSURKekNDQWcrZ0F3SUJBZ0lDQm<REDACTED>
    server: https://46230e4-1386-dsfklj-dfkj-4ed.k8s.ondigitalocean.com
  name: doks
contexts:
- context:
    cluster: doks
    user: $USER
  name: doks
current-context: doks
kind: Config
preferences: {}
users:
- name: $USER
  user:
    client-certificate: /Users/prabeshthapa/certs/$USER.crt
    client-key: /Users/prabeshthapa/certs/$USER.key
EOF
```

## Test the access
```
kubectl --kubeconfig=/Users/prabeshthapa/certs/config-prabesh-thapa cluster-info

Kubernetes master is running at https://2csdf8424-a23b2-445eb-128d-sdfkj.k8s.ondigitalocean.com
CoreDNS is running at https://fdf-gh-gh-gh-jh.k8s.ondigitalocean.com/api/v1/names
```