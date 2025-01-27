---
layout:     post
title:   "Integrate Grafana with GitHub OAuth"
date:       2025-01-06 15:13:18 +0200
image: grafana.png
tags:
    - devops
    - argocd
categories: DevOps
---

<p>Grafana ဆိုတာ data visualization တွေ monitoring တွေအတွက်သုံးတဲ့ open source project တစ်ခုပဲဖြစ်ပါတယ်။ UI ထဲကို login ဝင်ဖို့ဆို user account တွေကနေ ဝင်နိုင်သလို တစ်ခြား authentication methods တွေဖြစ်တဲ့ github , gitlab ကနေလည်းဝင်နိုင်ပါတယ်။ ဒီနေ့မှာ တော့ github oauth integration အကြောင်းကိုရေးပေးပါမယ်။ </p>

<h2>Registering an OAuth App on GitHub</h2>

<p>ပထမဆုံးအနေနဲ့ github ouath app တစ်ခုကို create လုပ်ပေးရပါမယ်။ Developer Settings ထဲကနေ OAuth app မှာ အောက်ကလို မိမိတို့ရဲ့ grafana url နဲ့ callback url ကိုထည့်ပေးလိုက်ပါ။ ပီးရင်တော့ save လိုက်မယ်ဆိုရင် client_id နဲ့ client_secret ကိုရလာပါလိမ့်မယ်။</p>

![oauth](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/oauth.png)

<h2>Update Helm Chart Values</h2>

<p>ပြီးသွားရင်တော့ ရလာတဲ့ client_id နဲ့ client_secret ကို helm value ရဲ့ grafana.iniထဲမှာ update ပေးလိုက်ပါ။</p>

```yaml
grafana:
  enabled: true
  grafana.ini:
    server:
      root_url: https://mydomain.com
    auth:
      disable_login_form: false
      disable_signout_menu: false
      auth.github:
        enabled: true
        allow_sign_up: true
        client_id: **************
        client_secret: **********
        scopes: user:email,read:org
        auth_url: https://github.com/login/oauth/authorize
        token_url: https://github.com/login/oauth/access_token
        api_url: https://api.github.com/user
  extraEnv: []
```

<h2>Configure Authentication in Grafana</h2>

<p> helm chart update ပြီးသွားရင်တော့ grafana ထဲကို login ဝင်ပြီး ​​​Administration >> Users and Access >> Authentication အောက်မှာ github ထဲကိုဝင်ပြီး အောက်ကလို client_id နဲ့ client_secret ထည့်လေးလိုက်ရင်ရပါပြီ။ </p>

![githubauth](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/github_auth.png)

<h2> Login via Github </h2>

ခဏနေရင် grafana user ကနေ logout ပြန်ထွက်လိုက်ပါ။ ဒါဆိုရင်တော့ အောက်ပါအတိုင်း Sign in with Github ဆိုတာတွေ့ရမှာဖြစ်ပြီးမိမိတို့ရဲ့ github acc နဲ့ login ဝင်နိုင်ပါပြီ။

![grafana](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/grafana.png)

အားလုံးကို ကျေးဇူးတင်ပါတယ်ခင်ဗျာ။






