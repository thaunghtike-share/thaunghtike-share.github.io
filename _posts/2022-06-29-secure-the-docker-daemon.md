---
layout:     post
title:   "Secure The Docker Daemon"
date:       2022-06-28 15:13:18 +0200
image: docker.png
tags:
    - Docker
categories: Docker
---

ကျွန်တော် ဒီနေ့ မှာတော့ Secure The Docker Daemon ဆိုတဲ့ ခေါင်းစဥ်အောက်မှာနားလည်သလောက်ရှင်းပြပေးသွားမှာဖြစ်ပါတယ်။ Secure လုပ်တာကိုမဆွေးနွေးခင်မှာ Docker Architecture အကြောင်းကိုအနည်းငယ်ပြန်စလိုက်ရအောင်ဗျာ။

<h2> What is docker ? </h2>

Docker ဆိုတာ ကျွန်တော်တို့ရဲ့ application တွေကို containerized လုပ်ပြီး ship လုပ်ဖို့ run ဖို့အတွက် သုံံးတဲ့ platform တစ်ခုပါ။ ကျွန်တော်တို့ရဲ့ application တစ်ခုချင်းစီ run ဖို့အတွက် container လို့ခေါ်တဲ့ environment တစ်ခုကို create လုပ်ပေးသွားမှာဖြစ်တယ်။ 

<h2> Docker Architecture </h2>

Docker က client-server architecture ပုံစံအလုပ်လုပ်ပါတယ်။ Docker daemon က server-side ဖြစ်ပြီး users တွေခေါ်လိုက်တဲ့ docker client cli ကနေတစ်ဆင့် host ပေါ်မှာ containers တွေ image တွေ volumes တွေနဲ့ဆိုင်တဲ့ operations ကို manage လုပ်ပေးသွားပါတယ်။ Docker client က system တစ်ခုတည်းမှာရှိနေတဲ့ docke daemon ကို interact လုပ်နိုင်သလို remote system ပေါ်က docker daemon ကိုလည်း interact လုပ်နိုင်ပါတယ်။

[docker architecture)()

Docker daemon နဲ့ client ဟာ REST API တွေကိုအသုံးပြုပြီး unix socket ကနေ communicate လုပ်ပါတယ်။ ပုံမှန်အားဖြင့် docker daemon ဟာ unix socket တစ်ခုကို /var/run/docker.sock မှာ create လုပ်ပေးပါတယ်။ အဲ့ဒီ socker ပေါ်မှာ client နဲ့ daemon က communicate လုပ်သွားတာပါ။ 

<h2> Acess the docker daemon remotely </h2>

ဒီနေရာမှာ client နဲ့ daemon ဟာ system တစ်ခုတည်းမှာဆို ပြသနာမရှိပါဘူး။ remote sytem က client တစ်ခုနေ access လုပ်ဖို့လိုလာပြီဆို TCP socket တစ်ခုကို docker daemon မှာ create ပေးရပါမယ်။ Daemon ကို access ရသွားတဲ့သူတိုင်းဟာ container တွေကို ကြိုက်သလို manage လုပ်ဖို့ အခွင့်ရေးရသွားပါတယ်။ container တွေ volumes တွေကို ဖျက်နိုင်တယ်။ privileged container ကိုသုံးပြီး root system ကိုပါ access ရသွားမှာပါ။ 

ဒါကြောင့် TCP socket ကို create တဲ့အခါမှာ docker daemon ကို secure ဖြစ်ဖို့လိုအပ်လာပါတယ်။ secure ဖြစ်ဖို့ဆို tls encryption အတွက်လိုအပ်တဲ့ CA တွေ certificate တွေကို create ရပါမယ်။ tls encrypt သာမလုပ်ဘူးဆိုရင် docker daemon ရဲ့ host ကိုသီတဲ့ ဘယ်သူမဆို access ပေးသလိုဖြစ်သွားမှာပါ။ Docker ရဲ့ port က 2375 ဖြစ်ပြီး tls ကိုသုံးထားရင်တော့ 2376 ဖြစ်ပါတယ်။

