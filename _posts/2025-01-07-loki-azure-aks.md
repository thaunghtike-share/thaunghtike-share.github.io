---
layout:     post
title:   "Deploy the Loki Helm chart on Azure"
date:       2025-01-07 15:13:18 +0200
image: loki.png
tags:
    - devops
    - argocd
categories: DevOps
---

<p>ကျွန်တော်ဒီနေ့မှာတော့ Loki ကို Azure AKS ပေါ်မှာ azure blob storage ကို backend အဖြစ် သုံးပြီးအသုံးပြုပုံလေးကိုရှင်းပြပေးပါမယ်။ Loki ဆိုတာ lightweight logging solution တစ်ခုဖြစ်ပြီး ပုံမှန်ဆိုရင် server ရဲ့ disk storage ကိုသုံးတာမလို့ storage ပြည့်သွားရင် ပြသနာတွေဖြစ်လာနိုင်ပါတယ်။ အဲ့ဒါမလို့ ​Azure Blob တို့ AWS S3တို့လို storage backend တစ်ခုကိုသုံပေးသင့်ပါတယ်။ ဒါကြောင့် ကျွန်တော်က azure blob ကိုသုံးပြီး ဒိနေ့ရှင်းပြပေးသွားပါမယ်။

Loki ကို azure နဲ့ authenticate လုပ်ဖို့ နည်းလမ်း(၃)ခုရှိပါတယ်။

-   Hard coding a connection string
-   Manged identity
-   Federated token

ကျွန်တော်ဒီနေ့မှာတော့ Federated Tokenကိုသုံးပြီး Loki ကို AKS ပေါ်မှာ deploy လုပ်တာကိုတစ်ဆင့်ချင်းရှင်းပြပေးသွားမှာဖြစ်ပါတယ်။</p>

