---
layout:     post
title:   "Setting Up TLS Encryption On Amazon EKS With ALB Ingress Controller"
date:       2021-09-15 15:13:18 +0200
image: 30.jpg
tags:
    - Kubernetes
    - EKS
categories: kubernetes
---   

<h2> Introduction </h2>

In this tutorial, I' gonna show you how to set up tls encryption on eks cluster with alb ingress controller. I already wrote a post about ssl nginx controller controller on my blog. You can read it [here](https://thaunghtike-share.github.io/2021/07/30/letencrypt-ssl-gke). In order for the Ingress resource to work, the cluster must have an ingress controller running. There are a lot of ingress controllers such as nginx, traefik and alb.

<h2> Prerequities </h2>

<ul>
    <li> eksctl </li>
    <li> eks cluster </li>
    <li> public hosted zone <li>
    <li> domain </li>
</ul>

<h2> Install Eksctl Binary </h2>

Firstly, we have to install eksctl binary to perform eks cluster operaions.

```bash
$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
$ sudo mv /tmp/eksctl /usr/local/bin
$ eksctl version
```
