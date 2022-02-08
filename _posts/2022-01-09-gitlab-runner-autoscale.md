---
layout:     post
title:   "Gitlab Runner Autoscaling With AWS Spot Instances"
date:       2022-01-09 15:13:18 +0200
image: 35.png
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

So why do we need to autoscale GitLab runners? Normally we register runners using Docker executor on another machine that’s separate from the one that hosts the GitLab instance for security and performance reason. It is not an issue when we run a small number of jobs. But when we run multiple jobs by multiple engineers parallelly, this machine which is used to register runner can't work well due to CPU overloading. Using a lot of machines to register gitlab runners does not seems a good option in technical view. If we are also using cloud service provider instances, why would you cost much for VMs monthly? We don't need the runners to active for the whole time. We can solve that problem by autoscaling AWS spot instances to install GitLab runners. Spot instances are almost 70-90% cheaper than normal instances. You can run it whenever you want and it will be terminated after processing the job automatically. Spot instances are used especially when we run batch jobs.

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

After installed all the necessary tools, it is time to register a runner. By registering a runner, we establish a connection between our gitlab host and our runner manager. There are various ways to register runners in gitlab, it all depends on your use case. Runners can be registered on a project level, or group level. Group level runners are available for all projects in the group, while project specific runners are just for a single repository. Select the project or group, navigate to Settings >> Runners, expand the runners section, scroll down and grab the registration token shown.

![22runtoken](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/2022runtoken.png)

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
We will be asked to enter some information, fill in the options as follows.

- gitlab cordinator url: this is your gitlab host url. If you are running a dedicated gitlab instance, enter the host url, otherwise use https://gitlab.com.

- gitlab ci token: Enter the token you just obtained

- gitlab runner description: Enter a description for the runner. Something like what the runner does should be fine

- gitlab ci tags: Tags are ways to say, run only jobs that has this tags on them. If this is not what you need, most likely, leave it empty. We can still have it   tagged but disable it later in the gitlab ui by setting the run untagged jobs option to true.

- runner executor:: For the runner executor. Make sure to enter docker+machine.

- default docker image: Specify your default docker image, when a job in gitlab-ci.yml file does not specify an image, this default image will be used.

<h1> Configure the Runner </h1>

Having registered the runner, it is time to configure the autoscaling features. This is where the final work is. Note also that you can register multiple runners by following the steps above. All you need is to get the targt project or group registration token.

In ubuntu, gitlab-runner configuration settings are saved and located at /etc/gitlab-runner/config.toml. Ssh once again into the ubuntu instance and fire up nano editor on this file. Here we will configure how the runner ochestrates and spins up and down aws ec2 spot instances.

```bash
sudo nano /etc/gitlab-runner/config.toml
```
Note: The gitlab runner must have network connection with every ec2 instance that it needs to create. To ensure that this need is met, we need to launch ec2 instances within the same vpc, utilizing any of the subnet network groups.

Once you fire up the editor, you will see an entry for the registered runner. This was created by gitlab when we first ran gitlab-runner register. We will edit this default configuration to our taste.

```yaml
concurrent = 4
check_interval = 0

[session_server]
  session_timeout = 1800
  
[[runners]]
  name = "ip-172-31-0-224"
  url = "https://gitlab.com"
  token = "hps5Dk5y9x6QwzmBqUV1"
  executor = "docker+machine"
  limit = 4
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = true
    disable_cache = true
    shm_size = 0
  [runners.cache]
    Type = "s3"
    Shared = true
    [runners.cache.s3]
      ServerAddress = "s3.amazonaws.com"
      AccessKey = "Your Access Key"
      SecretKey = "Your Secret Key"
      BucketName = "tho-s3-demo"
      BucketLocation = "us-east-1"
  [runners.machine]
    MachineDriver = "amazonec2"
    MachineName = "gitlab-ci-machine-%s"
    OffPeakTimezone = ""
    OffPeakIdleCount = 0
    OffPeakIdleTime = 0
    IdleCount = 0
    MachineOptions = [
      "amazonec2-access-key=*****",
      "amazonec2-secret-key=**********",
      "amazonec2-region=us-east-2",
      "amazonec2-vpc-id=vpc-9fec8ef4",
      "amazonec2-subnet-id=subnet-af179dc4",
      "amazonec2-zone=a",
      "amazonec2-use-private-address=true",
      "amazonec2-tags=gitlab-runner-autoscaler,gitlab,group-runner",
      "amazonec2-security-group=kube",
      "amazonec2-instance-type=t2.micro",
      "amazonec2-request-spot-instance=true",
      "amazonec2-spot-price=0.004"
    ]

``` 
<h3> Global Section </h3>

the global section defines rules that applies to all runners. check_interval defines the time interval in seconds at which gitlab runner communicates with the gitlab host to check for new jobs. defaults to 3 if not given. concurrent defines the maximum number of jobs that can be run concurrently by all runners put together. 0 means unlimited.

<h3> Runners Section </h3>

Each runner you register is listed in the [[runners]] section. This is what we see above. There are different executor types for gitlab-runner, but for us, we are interested in docker+machine, specified during the registration process.

limit defines the number of jobs that can be handled concurrently by the runner (also called token by gitlab). It can be equal to or less than the concurrent global option.

<h3> Runners.docker Section </h3>

This section configures the docker container. We disable volume cache since this will never help us because docker volume is lost once the build completes and ec2 instance is shut down. We will specify s3 as our cache location in the next section. [disable_cache] disables docker volume cache.
Runners.

<h3> Cache Section </h3>

Here we configure how the runner handles cache. Caching is necessary to speed up our jobs. We will be using s3 for cache, since the docker volume gets deleted once a job completes. We disable volume cache, and instead, specify s3 as our cache location.

We also provide necessary access and secret keys, as well as bucket name and location to the runner. Here, you specify the access key and secret key you created earlier in the tutorial. Also create a bucket. Name can be cache.your-domain.com.

The Shared configuration is very important, as this enables/disables cache sharing between runners.

<h3> Runners.machine Section </h3>

Here we configure the machine to be used for running the jobs. Most of the sections are self explanatory. But here are the key notes. Provide the previously created aws access and secret keys for amazonec2-access-key and amazonec2-secret-key respectively.

amazonec2-region specifies the region where the ec2 instance will be setup. Like I said earlier, take your time to study aws spot instances available in their various regions, and the bidding price using the link here.

<h3> Note on Networking </h3>

The runner manager instance (gitlab in the t.micro instance) needs to have network access to the region where the machines will be provisioned. The network configuration fields, (amazonec2-vpc-id, amazonec2-subnet-id, amazonec2-zone and amazonec2-security-group) are specifically for this. Here we setup the networking portion for our spot instances.

<h1> Running CICD pipeline in gitlab </h1>

So I will go to the demo project in gitlab and create a .gitlab-ci.yml file as shown below.

![ci](https://raw.githubusercontent.com/thaunggyee/thaunggyee.github.io/master/img/ci.png)

A job will be running in the pipeline soon. If the pipeline is pending, run command 'gitlab-runner –debug run ' on runner manager. Once in the running state, you can access the runner logs. When the job is done you will see the following

![job22running](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/job22running.png)

Also you will see one ec2 spot instance with size t2.micro running as below 

![ec2running](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/ec2spotrunning.png)

Here you can use any type you like, not just t2.micro. If you want to modify it, you can not just modify the type. For example, let's say you have a lot of jobs to run. We will convert from t2.micro to m4.xlarge. To do so, you will need to make two changes in config.toml. amazonec2-instance-type and amazonec2-spot-price. How to make a change? Look at the pricing on the spot instance. Compare the two pictures below.

![t2microprice](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/t2microprice.png)

![m4xlargeprice](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/m4xlargeprice.png)

In the first picture, the hourly price for t2.micro is $ 0.0035. In the next picture, the pricing for the m4.xlarge will be $ 0.040. So if we only use m4.xlarge, we need to set amazonec2-spot-price in the config to 0.05. Simply adjust the pricing to the instance type you want to use.

The pipeline will be passed after a couple of minutes.

![22pipepass](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/22pipepass.png)

If you go back to the spot instance when the pipelie is passed, you will see that it is closed and the status is terminated.

In this way, it will create a spot instance for each pipeline and it will scale in automatically when the pipeline is completed. Also you do not need to spend a lot of money. 
 
References

- https://blog.dopevs.cloud/2019-08-02-Gitlab-Runner-Autoscaling-with-aws-spot-instances
- https://gitlab.com
- https://medium.com/@dev.npalm/gitlab-runners-on-the-aws-spotinstances-c637503e3c17

Thanks for your time !!! !
