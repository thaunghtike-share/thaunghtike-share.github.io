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


                      
