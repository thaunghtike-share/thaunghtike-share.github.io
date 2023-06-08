---
layout:     post
title:   "Argocd Image Updater"
date:       2023-06-08 15:13:18 +0200
image: argocdimageupdater.png
tags:
    - devops
    - argocd
categories: DevOps
---

<p>စာမရေးဖြစ်တာလည်း အတော်လေးကြာသွားပါပြီ။ ဒိနေ့တော့ ကျွန်တော် Argo CD အကြောင်းလေးနဲ့ပြန်လည်အစပြုလိုက်ပါမယ်။ DevOps Engineer တော်တော်များများ argocd ကတော့ရင်းနှီးပြီးသားဖြစ်မယ်ထင်ပါတယ်။ Topic တွေအများကြီးထဲကမှ Argo CD Image Updater အကြောင်းကို နားလည်သလောက် ရေးသွားပါမည်။ </p>

<h2>👉 Introduction</h2>

<p>တစ်နည်းအားဖြင့် image updater ဆိုတာ kubernetes workload တွေမှာအသုံးပြုထားတဲ့ container images တွေကို automatically update လုပ်ပေးသွားတဲ့ tool တစ်ခုလည်းဖြစ်ပါတယ်။ </p>

> **_NOTE:_**
A tool to automatically update the container images of Kubernetes workloads that are managed by Argo CD. The Argo CD Image Updater can check for new versions of the container images that are deployed with your Kubernetes workloads and automatically update them to their latest allowed version using Argo CD.


<p> ဆိုလိုရင်းကတော့ container registry ထဲမှာ image tag ဒါမှမဟုတ် digest အသစ်တစ်ခုရောက်လာတာနဲ့ ွ GitOps repo ထဲမှာ image အသစ် နဲ့ update လုပ်ပေးသွားမှာဖြစ်ပါတယ်။

image updater မပေါ်ခင်တုန်းကတော့ CICD မှာ script တွေရေးကြပြီး image အသစ်ကို gitops repo ထဲမှာ update ဖြစ်အောင်လုပ်ခဲ့ကြတာပေါ့။ ဒါဆို အနည်းငယ်တော့ သဘောပေါက်လောက်ပြီထင်တယ်။ အသေးစိတ်ကိုအောက်မှာ ဆက်လက်ဖတ်ရှုပေးကြပါ။ </p>



