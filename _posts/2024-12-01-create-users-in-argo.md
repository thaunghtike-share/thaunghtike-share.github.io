---
layout:     post
title:   "Create ArgoCD Users & Assign RBAC on Kubernetes"
date:       2024-12-01 15:13:18 +0200
image: argoaaa.png
tags:
    - devops
    - argocd
categories: DevOps
---

<p> လိုအပ်ရင် ကိုယ်တိုင်ပြန်သုံးဖို့ရော လိုအပ်တဲ့သူတွေရော သုံးနိုင်ဖို့ ဒီနေ့ argocd မှာ user တွေ role တွေသတ်မှတ်ပုံကို ကျွန်တော် သိသလောက်ရေးလိုက်ပါတယ်။ ArgoCD ဆိုတာ Kubernetes ပေါ်မှာ GitOps အတွက်သုံးတဲ့ Continuous Delivery Tool တစ်ခု ဖြစ်ပါတယ်။ ပုံမှန်ဆိုရင် argocd server မှာ admin user ကို default ထည့်ပေးလိုက်ပါတယ်။ တစ်ခြား user တွေထပ်ထည့်ချင်တယ်ဆိုရင်တော့ UI ကနေ လုပ်လို့ရတဲ့ feature မပါပါဘူး။ </p>

<p> argocd namespace ထဲမှာရှိတဲ့ configmap တွေကို list လုပ်ကြည့်လိုက်ပါ။</p>

```
$ kubectl get cm -n argocd
NAME                              DATA   AGE
argocd-cm                         2      4d5h
argocd-cmd-params-cm              1      4d5h
argocd-gpg-keys-cm                0      4d5h
argocd-image-updater-config       2      3d6h
argocd-image-updater-ssh-config   0      3d6h
argocd-notifications-cm           0      4d5h
argocd-rbac-cm                    1      4d5h
argocd-ssh-known-hosts-cm         1      4d5h
argocd-tls-certs-cm               0      4d5h
```
အဲ့ထဲကမှ argocd-cm နဲ့ argocd-rbac-cm တို့က role နဲ့ policy တွေ သတ်မှတ်ပေးရမယ့် configmap တွေဖြစ်ကြတယ်။ အရင်ဆုံး အောက်ပါအတိုင်း configmap တွေကို yaml အဖြစ်သိမ်းထားလိုက်ပါ။

```
kubectl get cm argocd-cm -n=argocd -o yaml > argocd-cm.yaml
kubectl get cm argocd-rbac-cm -n=argocd -o yaml > argocd-rbac-cm.yaml
```
ပြီးရင်တော့ ကိုယ်ထည့်ချင်တဲ့ users တွေကို argocd-cm.yaml မှာ အောက်ကလိုထည့်လိုက်ပါ။

```
apiVersion: v1
data:
  accounts.minko: login
  accounts.wailwin: login
kind: ConfigMap
```
ဒါဆိုရင် kubectl apply နဲ့ configmap ကို update ပေးလိုက်ပါ။
```
kubectl apply -f argocd-cm.yaml
```
နောက်တစ်ဆင့်ကတော့ role တွေသတ်မှတ်ပီး attach လုပ်ပေးဖို့အတွက် argocd-rbac-cm ကို ပြင်ပေးဖို့လိုပါတယ်။ ပုံမှန်တော့ readwrite, readonly, readexecute ဆိုတဲ့ role တွေကအသုံးများပြီး အဲ့ထဲမှ RW, RO တို့နဲ့စမ်းပြလိုက်ပါတယ်။

```
apiVersion: v1
data:
  policy.csv: |
    p, role:readwrite-user, applications, get, *, allow
    p, role:readwrite-user, applications, sync, *, allow
    p, role:readwrite-user, applications, create, *, allow
    p, role:readwrite-user, applications, update, *, allow
    p, role:readwrite-user, applications, delete, *, allow
    p, role:readwrite-user, applications, patch, *, allow
    p, role:readwrite-user, applications, rollback, *, allow
    p, role:readwrite-user, applications, terminate, *, allow
    p, role:readonly-user, applications, get, *, allow
    p, role:readonly-user, applications, sync, *, allow
    g, minko, role:readwrite-user
    g, wailwin, role:readonly-user
kind: ConfigMap
```
ကျွန်တော်ကတော့ argocd project အကုန်လုံးကို wildcard ပေးထားတာမလို့ ဖတ်ရတာ ရိုးရှင်းနိုင်ပါတယ်။ တကယ်လို့ project တစ်ချို့ကိုပဲ permission ပေးဖို့လိုအပ်လာရင်တော့ <project_name>/<app_name> စသည်ဖြင့်သတ်ဆိုင်ရာ applications တွေကိုပဲ assign ချနိုင်ပါတယ်။

<p> နောက်ဆုံးအဆင့်ကတော့ user တွေအတွက် password သတ်မှတ်ပေးရပါမယ်။ argocd binary ကို download ပီး login ဝင်ပေးရပါမယ်။</p>

```
argocd login argocd.mydomain.com:443 --skip-test-tls --grpc-web --username admin --password *************
'admin:login' logged in successfully
Context 'argocd.mydomain.com:443' updated
```
login ပြီးပြီဆိုရင်တော့ မိမိတို့ သတ်မှတ်ပေးချင်တဲ့ user ကို password အသစ်လုပ်ပေးလိုက်ပါ။ current-password နေရာမှာ admin password ကိုပဲသုံးပါ။

```
 argocd account update-password --account minko --current-password ***** --new-password ******
```
ဒါဆို argocd server မှာ user အသစ်တွေနဲ့ login လို့ရပါပြီ။ အားလုံးကို ကျေးဇူးတင်ပါတယ်ခင်ဗျာ 
