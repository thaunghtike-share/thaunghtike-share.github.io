---
layout:     post
title:   "Install Sonarqube On Kubernetes EKS cluster"
date:       2022-06-12 15:13:18 +0200
image: 36.png
tags:
    - Kubernetes
categories: Kubernetes
---

ကျွန်တော်ဒီနေ့မှာတော့ Sonarqube ကို EKS cluster ပေါ်မှာ install လုပ်တဲ့အကြောင်းကို ရှင်းပြပေးမှာပါ။ Sonarqube ဆိုတာက ကျွန်တော်တို့ ရဲ့ application code တွေထဲက bug တွေ vulnerabilities တွေကို automatic detect လုပ်ဖို့အတွက် သုံးတာပါ။ Open Source project ဖြစ်ပြီး language language တော်တော်များများနဲ့ integrate လုပ်ပြီးအသုံးပြုနိုင်ပါတယ်။

<h2> Create EKS Cluster using eksctl </h2>

အရင်ဆုံး eksctl နဲ့ cluster တစ်ခုကို အောက်ကအတိုင်း create လိုက်ပါမယ်။ eksctl ဆိုတာ eks cluster တွေကို manage လုပ်ဖို့သုံးတာပါ။ ကိုယ့်စက်ထဲမှာ install မလုပ်ရသေးရင်တော့ [ဒီက](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html) နေဖတ်ပြီးလုပ်နိုင်ပါတယ်။

```bash
~ % eksctl create cluster --name=eksdemo \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup
```

<h2> Create & Associate IAM OIDC Provider for our EKS Cluster </h2>

cluster ပြီးသွားရင်တော့ OIDC provider ကို create ပေးရပါမယ်။ EKS ပေါ်မှာသုံးမယ့် service account တွေကို IAM role တွေနဲ့ enable လုပ်နိုင်ဖို့အတွက် OIDC provider တစ်ခုကို အောက်ပါအတိုင်း create လုပ်ပေးလိုက်ပါ။

```bash
~ % eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster eksdemo \
    --approve
```

<h2> Create Node Group with additional Add-Ons in Public Subnets </h2>

ပြီးရင်တော့ eks cluster မှာသုံးဖို့ node group တစ်ခု create လိုက်ပါမယ်။

```bash
eksctl create nodegroup --cluster=eksdemo \
                       --region=us-east-1 \
                       --name=eksdemo1-ng-public1 \
                       --node-type=t3.medium \
                       --nodes=3 \
                       --nodes-min=3 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access
```                       

kubeconfig file ကို ~/.kue/config ထဲကို ထည့်ပါလိုက်မယ်။ ပြီးသွားရင်တော့ kubectl get nodes နဲ့ကြည့်လိုက်ရင် worker ၃လုံး ready ဖြစ်နေတာကိုတွေ့ရပါလိမ့်မယ်။

```bash
~ % kubectl get nodes
NAME                             STATUS   ROLES    AGE    VERSION
ip-192-168-28-242.ec2.internal   Ready    <none>   107m   v1.21.12-eks-5308cf7
ip-192-168-52-205.ec2.internal   Ready    <none>   107m   v1.21.12-eks-5308cf7
ip-192-168-7-38.ec2.internal     Ready    <none>   107m   v1.21.12-eks-5308cf7
```

                      
