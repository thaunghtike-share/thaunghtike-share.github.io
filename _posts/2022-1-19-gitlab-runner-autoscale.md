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

<h1> Why Runner Needs Autoscaling </h1>

So why do we need to autoscale GitLab runners? ။ Normally we register runners using Docker executor on another machine that’s separate from the one that hosts the GitLab instance for security and performance reason. It is not an issue when we run a small number of jobs. But when we run multiple jobs by multiple engineers parallelly, this machine which is used to register runner can't work well due to CPU overloading. Using a lot of machines to register gitlab runners does not seems a good option in technical view. If we are also using cloud service provider instances, why would you cost much for VMs monthly? We don't need the runners to active for the whole time. We can solve that problem by autoscaling AWS spot instances to install GitLab runners. Spot instances are almost 70-90% cheaper than normal instances. You can run it whenever you want and it will be terminated after processing the job automatically. Spot instances are used especially when we run batch jobs.  

<h1> Runner Manager (or) Bastion Host </h1>

Runner manager is used to do scale in (or) scale out spot instances. Firstly we should create a user who has Ec2 and S3 full access. I will use aws root user account in this demo. So, I will not create any IAM user. It's enough t2.small instance for runner manager. I already created an ec2 instance for manager. Let's check the instance.

```bash
thaunghtikeoo@thaunghtikeoo:~$ aws ec2 describe-instances --query "Reservations[*].Instances[*].{name: Tags[?Key=='Name'] | [0].Value, instance_id: InstanceId, ip_address: PrivateIpAddress, state: State.Name}" --output table
--------------------------------------------------------------------------------------------
|                                     DescribeInstances                                    |
+----------------------+----------------+------------------------------------+-------------+
|      instance_id     |  ip_address    |               name                 |    state    |
+----------------------+----------------+------------------------------------+-------------+
|  i-01559252cc9d7baf8 |  172.31.81.166 |  runner-manager                    |  running    |
+----------------------+----------------+------------------------------------+-------------+
```
Then we have to create a S3 bucket to store runner config and cache. Create a s3 bucket.

```bash

thaunghtikeoo@thaunghtikeoo:~$ aws s3 mb s3://tho-s3-demo
make_bucket: tho-s3-demo
```
<h1> Register gitlab runner </h1>

After creating ec2 instance and s3 bucket, ssh into ec2 instance. Install gitlab runner binary , docker and also docker machine.

NOTE - docker-machine has been depricated.

```bash

# gitlab ruunner binary install 

sudo curl -L --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64"
sudo chmod +x /usr/local/bin/gitlab-runner

# docker install 

sudo su - && apt update -y && apt install docker.io -y && systemctl restart docker

# docker-machine install 

base=https://github.com/docker/machine/releases/download/v0.16.0 \
  && curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine \
  && sudo mv /tmp/docker-machine /usr/local/bin/docker-machine \
  && chmod +x /usr/local/bin/docker-machine
  
```  
<h1> Register a Runner </h1>

Having installed all the necessary tools, it is time to register a runner. By registering a runner, we establish a connection between our gitlab host and our runner manager. There are various ways to register runners in gitlab, it all depends on your use case. Runners can be registered on a project level, or group level. Group level runners are available for all projects in the group, while project specific runners are just for a single repository. Select the project or group, navigate to Settings >> Runners, expand the runners section, scroll down and grab the registration token shown.

![22runtoken](22runtoken.png)

With the token at hand, ssh once again to the instance, lets register the runner by running the interactive command:

```bash
root@ip-172-31-81-166:~# gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=15351 revision=98daeee0 version=14.7.0
Running in system-mode.                            
                                                   
Enter the GitLab instance URL (for example, https://gitlab.com/):
https://gitlab.com/
Enter the registration token:
jDQCx1yfZb9paXQowCex
Enter a description for the runner:
[ip-172-31-81-166]: 
Enter tags for the runner (comma-separated):

Registering runner... succeeded                     runner=jDQCx1yf
Enter an executor: docker-ssh, parallels, docker, shell, ssh, virtualbox, docker+machine, docker-ssh+machine, kubernetes, custom:
docker+machine
Enter the default Docker image (for example, ruby:2.6):
alpine:latest
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```

