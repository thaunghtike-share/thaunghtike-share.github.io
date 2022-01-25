---
layout:     post
title:   "Gitlab Runner Autoscaling With AWS Spot Instances"
date:       2022-01-19 15:13:18 +0200
image: 35.jpg
tags:
    - CICD
    - AWS
categories: CICD
---

<h1> Introduction </h1>

Hi guys, today I will talk about how to setup gitlab runner autoscaling using AwS spot instances. I already made a post for gitlab runner autoscaling using GCP preemptible vms. You can read it [here](https://blog.thaunghtikeoo.info/2021/07/14/gitlab-runner-gcp). There is no differences between preemptible VMs and aws spot instances. They are the same concepts. So I will only write to set up labs. 

<h1> What is a gitlab runner </h1>

GitLab Runner is an application that works with GitLab CI/CD to run jobs in a pipeline.You can choose to install the GitLab Runner application on infrastructure that you own or manage. If you do, you should install GitLab Runner on a machine that’s separate from the one that hosts the GitLab instance for security and performance reasons. When you use separate machines, you can have different operating systems and tools, like Kubernetes or Docker, on each. GitLab Runner is open-source and written in Go. It can be run as a single binary; no language-specific requirements are needed. You can install GitLab Runner on several different supported operating systems. Other operating systems may also work, as long as you can compile a Go binary on them. GitLab Runner can also run inside a Docker container or be deployed into a Kubernetes cluster. 

<h1> Why Runner Needs Autoscaling <h1>

So why do we need to autoscale GitLab runners? ။ Normally we register runners using Docker containers or separate machines. It is not an issue when we run a small number of jobs. But when we run multiple jobs by multiple engineers parallelly, this machine which is used to register runner can't work well due to CPU overloading. There are a lot of ways to solve that problem. We can either use the Kubernetes cluster or use AWS spot instances to install GitLab runners.
