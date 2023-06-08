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

<p>စာမရေးဖြစ်တာလည်း အတော်လေးကြာသွားပါပြီ။ ဒီနေ့တော့ ကျွန်တော် Argo CD အကြောင်းလေးနဲ့ပြန်လည်အစပြုလိုက်ပါမယ်။ DevOps Engineer တော်တော်များများ argocd ကတော့ရင်းနှီးပြီးသားဖြစ်မယ်ထင်ပါတယ်။ Topic တွေအများကြီးထဲကမှ Argo CD Image Updater အကြောင်းကို နားလည်သလောက် ရေးသွားပါမည်။ </p>

<h2>👉 Introduction</h2>

<p>တစ်နည်းအားဖြင့် image updater ဆိုတာ kubernetes workload တွေမှာအသုံးပြုထားတဲ့ container images တွေကို automatically update လုပ်ပေးသွားတဲ့ tool တစ်ခုလည်းဖြစ်ပါတယ်။ </p>

> A tool to automatically update the container images of Kubernetes workloads that are managed by Argo CD. The Argo CD Image Updater can check for new versions of the container images that are deployed with your Kubernetes workloads and automatically update them to their latest allowed version using Argo CD.


<p> ဆိုလိုရင်းကတော့ container registry ထဲမှာ image tag ဒါမှမဟုတ် digest အသစ်တစ်ခုရောက်လာတာနဲ့ GitOps repo ထဲမှာ image အသစ် နဲ့ update လုပ်ပေးသွားမှာဖြစ်ပါတယ်။ </p>

<h2>👉 Image Updater အကြောင်း</h2>

<p>image updater မပေါ်ခင်တုန်းကတော့ CICD မှာ script တွေရေးကြပြီး image အသစ်ကို gitops repo ထဲမှာ update ဖြစ်အောင်လုပ်ခဲ့ကြတာပေါ့။ ဒါဆို အနည်းငယ်တော့ သဘောပေါက်လောက်ပြီထင်တယ်။ အသေးစိတ်ကိုအောက်မှာ ဆက်လက်ဖတ်ရှုပေးကြပါ။

သူရဲ့ ကောင်းတဲ့အချက်က docker image တွေကို build and push လုပ်ပြီးတိုင်းမှာ ကျွန်တော်တို့ gitops repoမှာ image tag ကိုသွားပြောင်းပေးဖို့မလိုပါဘူး။ argocd image updater ကနေ အဲ့ဒါကို လုပ်ပေးသွားမှာဖြစ်ပါတယ်။ gitops repo ထဲမှာ မပြောင်းပဲ cluster ထဲကို တိုက်ရိုက် image update အောင်လည်း လုပ်လို့လည်းရပါတယ်။ ဒါကတော့ productionအတွက်မသင့်တော်ပါဘူး</p>

<h2>👉 Install လုပ်နည်း</h2>

Argocd ကို install ခဲ့တဲ့ namespace ထဲမှာပဲ image updater ကိုလည်း install ပေးဖို့လိုပါတယ်။ [ဒီမှာ](https://argocd-image-updater.readthedocs.io/en/stable/install/installation/) instruction တွေကြည့်ပြီး install နိုင်ပါတယ်။

```code
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```
ပြီးသွားတဲ့အခါမှာတော့ အောက်ကလို image updater pod တစ်ခု run နေတာကိုတွေ့ရပါလိမ့်မည်။

```bash
$ kubectl get pods -n argocd
NAME                                                READY   STATUS    RESTARTS       AGE
argocd-application-controller-0                     1/1     Running   0              3d23h
argocd-applicationset-controller-59dcb74c66-8g2bn   1/1     Running   0              7d1h
argocd-image-updater-947df9d9b-7khm9                1/1     Running   0              43h
argocd-notifications-controller-56b4b9db6f-kprlx    1/1     Running   0              7d1h
argocd-redis-74d77964b-6dl5q                        1/1     Running   0              3d23h
argocd-repo-server-6dfd684ff-knj8z                  1/1     Running   0              7d1h
argocd-server-676c6b849b-r6lk5                      1/1     Running   0              7d1h
```
ဒုတိယအဆင့်အနေနဲ့ log level ကို debug ပြောင်းပေးရပါမည်။ argocd-image-updater-config ဆိုတဲ့ configmap ကို edit ပေးရပါမည်။

> kubectl edit configmaps --namespace argocd argocd-image-updater-config

အောက်ပါအတိုင်း log level ကို သတ်မှတ်ပေးလိုက်ပါ။

```bash
apiVersion: v1
data:
  log.level: debug
kind: ConfigMap
```
<p> နောက်ဆုံးအဆင့်အနေနဲ့ argocd image updater ကို ကျွန်တော်တို့ရဲ့ container registry နဲ့ authenticate လုပ်ပေးရပါမယ်။ ခုဏ က configmap ကိုပဲ ပြန် edit ပြီး မိမိရဲ့ registry credentails တွေကိုထည့်ပေးပါ။ အသေးစိတ်ကို [ဒီမှာ](https://argocd-image-updater.readthedocs.io/en/stable/configuration/registries/) ကြည့်နိုင်ပါတယ်။

အရင်ဆုံး docker-registry secret တစ်ခု create ပေးရပါမည်။ ကျွန်တော်ကတော့ azure container registry ကိုသုံးထားပါတယ်။</p>

> kubectl -n argocd create secret docker-registry mycontainerregistry-secret --docker-server=mycontainerregistry.azurecr.io --docker-username=mycontainerregistry --docker-password=PASSWORD -o yaml --dry-run=client | kubectl -n argocd apply -f -

<p> credentials နေရာမှာ pull secret ကိုသုံးထားပါတယ် အောက်က argocd က namespace ဖြစ်ပြီး အနောက်ကတော့ registry ရဲ့ secret ဖြစ်ပါတယ်။ </p>

```bash
apiVersion: v1
data:
  log.level: debug
  registries.conf: |
    registries:
    - name: Azure Container Registry
      prefix: dinger.azurecr.io
      api_url: https://myreg.azurecr.io
      credentials: pullsecret:argocd/mycontainerregistry-secret
kind: ConfigMap
metadata:
```
> ဒီနေရာမှာ အရေးကြီးဆုံးတစ်ခုက argocd image updaer pod ကို restart လုပ်ပေးရပါမယ်။

<p>pod ကို kubectl delete နဲ့ ဖျက်လိုက်ပါ။ pod အသစ်တစ်ခုပြန်ထွက်လာပါလိမ့်မည်။ </p>

<h2>👉 Let's see how it works </h2>


