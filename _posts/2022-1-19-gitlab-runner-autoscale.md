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

So why do we need to autoscale gitlab runners? ။ ။ ပြသနာကအဲ့မှာ စတော့တာပါပဲ။ DevOps ၊ SRE team တွေမှာ လူနည်းရင်တော့ သိပ်ပြီးမသိသာပါဘူး။ CI CD pipeline တွေကို လူတွေအများကြီးက ပြိုင်တူ run တဲ့အခါ docker နဲ့ register လုပ်ထားတဲ့စက်က CPU overload ဖြစ်ပြီး ကောင်းကောင်းအလုပ်မလုပ်နိုင်တော့ပါဘူး။ runner တွေအများကြီး register လုပ်ခြင်းကလည်း technical ဘက်ကကြည့်ရင်သိပ်ပြီး နည်းစနစ်မကျပါဘူး။ ဒါဆို cloud ပေါ်က compute engine ကို scale up လုပ်ဖို့ဆိုရင်လည်း စဥ်းစားစရာဖြစ်လာပြန်ရော။ runner လေးတစ်ခုအတွက်ပဲသုံးမယ့် vm အတွက် ဘာလို့ တစ်လ ကို ပိုက်ဆံတွေအများကြီးပေးနေရမှာလဲ။ အဲ့ဒီအတွက် ဖြေရှင်းဖို့ အဖြေကတော့ spot instances တွေနဲ့ autoscale လုပ်ခြင်းပဲဖြစ်ပါတယ်။ spot instance ဆိုတာ normal instance တွေထက် 70-90% လောက်အထိ စျေးသက်သာပါတယ်။ ကိုယ်လိုတဲ့အချိန်မှာသာ run နိုင်ပြီး ပြီးသွားရင်ပြန်ပိတ်ခဲ့လို့ရပါတယ်။ အထူးသဖြင့် batch job တွေ run တဲ့အခါသုံးကြပါတယ်။
