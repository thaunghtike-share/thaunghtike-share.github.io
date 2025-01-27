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

<p>Grafanဆိုတာ data visualization တွေ monitoring တွေအတွက်သုံးတဲ့ open source project တစ်ခုပဲဖြစ်ပါတယ်။ UI ထဲကို login ဝင်ဖို့ဆို user account တွေကနေ ဝင်နိုင်သလို တစ်ခြား authenction methods တွေဖြစ်တဲ့ github , gitlab ကနေလည်းဝင်နိုင်ပါတယ်။ ဒီနေ့မှာ တော့ github oauth integration အကြောင်းကိုရေးပေးပါမယ်။ </p>

```yml
grafana:
  enabled: true
  grafana.ini:
    server:
      root_url: https://grafana.maharbawgammoney.com
    auth:
      disable_login_form: false
      disable_signout_menu: false
      auth.github:
        enabled: true
        allow_sign_up: true
        client_id: Ov23litwgJ42oRU74BiK
        client_secret: added106974882a075cf98743a07f841ff8cbcf9
        scopes: user:email,read:org
        auth_url: https://github.com/login/oauth/authorize
        token_url: https://github.com/login/oauth/access_token
        api_url: https://api.github.com/user
  extraEnv: []
```

<h2>Registering an OAuth App on GitHub</h2>

<p>>ပထမဆုံးအနေနဲ့ github ouath app တစ်ခုကို create လုပ်ပေးရပါမယ်။ Devloper Settings ထဲကနေ OAuth app မှာ အောက်ကလို မိမိတို့ရဲ့ grafana url နဲ့ callback url ကိုထည့်ပေးလိုက်ပါ။ ပီးရင်တော့ save လိုက်မယ်ဆိုရင် client_id နဲ့ client_secret ကိုရလာပါလိမ့်မယ်။</p>

![oauth](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/oauth.png)

<h2>Update Helm Chart Values</h2>

<p>ပြီးသွားရင်တော့ ရလာတဲ့ client_id နဲ့ client_secret ကို helm value ရဲ့ grafana.iniထဲမှာ update ပေးလိုက်ပါ။</p>

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

<h2>Configure Authentication in Grafana</h2>

<p> helm chart update ပြီးသွားရင်တော့ grafan ထဲကို login ဝင်ပြီး ​​​Administartion >> Users and Access >> Authentication အောက်မှာ github ထဲကိုဝင်ပြီး အောက်ကလို client_id နဲ့ client_secret ထည့်လေးလိုက်ရင်ရပါပြီ။ </p>

![githubauth](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/github_auth.png)





