---
layout:     post
title:      "Let's Encrypt SSL Certificate On Nginx Ingress Controller"
subtitle:   "How To Setup Let's Encrypt SSL Certificate On Nginx Ingress Controller For GEK"
date:       2021-07-30 4:00:00
author:     "Thaung Htike Oo"
image: 5.jpg
categories: kubernetes
catalog: true
tags:
    - Kubernetes
    - GKE
---

<h2> Introduction </h2>

ကျွန်တော်ဒီနေ့ရေးမှာကတော့ kubernetes ပေါ်မှာ ingress ကို cert-manager နဲ့ ssl certificate setup လုပ်တဲ့ အကြောင်းပဲဖြစ်ပါတယ်။ kubernetes ပေါ်မှာ service တွေကို cluster ပြင်ပ ကနေ access လုပ်ဖို့ဆို Nodeport ဒါမှမဟုတ်ရင် Loadbalancer အဖြစ် expose လုပ်ပေးဖို့လိုပါတယ်။ ingress ဆိုတာက အလွယ်ပြောရရင် အဲ့ဒီ service တွေကို domain နဲ့ ချိတ်ဆက်ပြီး သုံးဖို့အတွက်ပဲဖြစ်ပါတယ်။ ingress သီးသန့်သုံးမယ်ဆို service တွေဟာ secure မဖြစ်သေးပါဘူး။ ဒါကြောင့် secure ဖြစ်ဖို့ဆို cert-manager နဲ့ ssl certificate ထုတ်ပြီး ingress မှာ ထည့်ပေးရမှာပါ။ အခုပဲ lab ကိုစလိုက်ပါတော့မယ်။

<h2> Prepare A Kubernetes Cluster </h2>

kubernetes cluster တစ်ခုအရင် install လုပ်ရပါမယ်။ မိမိကြိုက်နှစ်သက်ရာ platform နဲ့ k8s cluster တစ်ခုကို တည်ဆောက်နိုင်ပါတယ်။ ကျွန်တော်ကတော့ gke cluster တစ်ခုကို create လိုက်ပါမယ်။ cluster ready ဖြစ်ပြီဆို kubectl get nodes နဲ့ gcp cloud shell ကနေစစ်ကြည့်လိုက်ပါ။

```bash
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ kubectl get nodes
NAME                                  STATUS   ROLES    AGE    VERSION
gke-demo-default-pool-70e88b74-2m58   Ready    <none>   5m5s   v1.20.8-gke.900
gke-demo-default-pool-70e88b74-5rjx   Ready    <none>   5m5s   v1.20.8-gke.900
gke-demo-default-pool-70e88b74-ff19   Ready    <none>   5m6s   v1.20.8-gke.900
```
<h2> Install Nginx Ingress Controller </h2>

ingress သုံးဖို့ဆို ingress controller တစ်ခုကို install အရင်ဆုံးလုပ်ပေးဖို့လိုပါမယ်။ Nginx ၊ Traefik ၊ Kong စသည်ဖြင့် controller တွေကတော့ အများကြီးရှိပါတယ်။ ကျွန်တော်ကတော့ nginx controller ကိုသုံးလိုက်ပါမယ်။ nginx controller ကို helm နဲ့ install လုပ်ပါမယ်။ helm install မလုပ်ရသေးရင် ubuntu မှာဆို snap install helm --classic နဲ့ install လုပ်နိုင်ပါတယ်။ helm ဆိုတာ kubernetes အတွက် package manager တစ်ခုပါ။ အရင်ဆုံး nginx ingress အတွက်လိုအပ်တဲ့ helm repo ကို add ပေးရပါမယ်။

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```
repo add ပြီးသွားရင်တော့ nginx ingress controller ကိုအောက်ကအတိုင်း install လုပ်နိုင်ပါတယ်။

```bash
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ helm install ingress-nginx ingress-nginx/ingress-nginx
NAME: ingress-nginx
LAST DEPLOYED: Sat Jul 31 12:30:05 2021
NAMESPACE: defaultSTATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w ingress-nginx-controller'
```
အခုဆိုရင် default namespace မှာ nginx controller run နေတာကို တွေ့ရမှာဖြစ်ပါတယ်။ 

```bash
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ kubectl get all
NAME                                           READY   STATUS    RESTARTS   AGE
pod/ingress-nginx-controller-d9458694b-srcss   1/1     Running   0          87s

NAME                                         TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.52.9.132   104.198.153.12   80:30230/TCP,443:32478/TCP   89s
service/ingress-nginx-controller-admission   ClusterIP      10.52.11.50   <none>           443/TCP                      89s
service/kubernetes                           ClusterIP      10.52.0.1     <none>           443/TCP                      15m

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           88s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-d9458694b   1         1         1       88s
```
nginx ingress controller run နေတာကို သိဖို့ loadbalancer ip ဖြစ်တဲ့ 104.198.153.12 ကို browser ကခေါ်ကြည့်ရင် အောက်ကအတိုင်းတွေ့ရမှာပါ။

![nginxcontroller]()

<h2> Create Sample Nginx Deployment And Service </h>

kubernetes service ကိုတော့ nginx နဲ့ပဲ demo စမ်းပြပါမယ်။ အောက်ကအတိုင်း nginx deployment နဲ့ service ကို create နိုင်ပါတယ်။ nginx service ကိုတော့ ingress စမ်းမှာဆိုတော့ loadbalancer အနေနဲ့ create ရပါမယ်။ replica က တစ်ခုကိုပဲသုံးပါမယ်။ service type က loadbalancer ပါ။ 

```bash
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx-app
  namespace: default
  labels:
    app: nginx-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: "nginx"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-app
  namespace: default
spec:
  selector:
    app: nginx-app
  ports:
  - name: http
    targetPort: 80
    port: 80
  type: LoadBalancer  
```    
kubectl create နဲ့ yaml file ကို create လိုက်ပါမယ်။

```bash
kubectl create -f nginx.yaml
```
cloud provider တွေဖြစ်တဲ့ gke ၊ aks ၊ eks တွေမှာဆို loadbalancer နဲ့ expose လုပ်တဲ့ service တွေက public ip တစ်ခု assign ချပေးမှာဖြစ်ပါတယ်။ အခုလည်း nginx service အတွက် public ip တစ်ခု ရလာပါတယ်။ 

```bash
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ kubectl get svc
NAME                                 TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.52.9.132   104.198.153.12   80:30230/TCP,443:32478/TCP   13m
ingress-nginx-controller-admission   ClusterIP      10.52.11.50   <none>           443/TCP                      13m
kubernetes                           ClusterIP      10.52.0.1     <none>           443/TCP                      28m
nginx-app                            LoadBalancer   10.52.1.182   35.184.183.163   80:32569/TCP                 40s
```
အဲ့ ip ကို browser ကနေ ခေါ်လိုက်ရင် nginx default page ကို တွေ့ရမှာပါ။ 

![nginx]()

<h2> Setup Ingress For Nginx Service </h2>

public ip နဲ့ access ရပြီဆိုတော့ domain နဲ့ ခေါ်သုံးဖို့ ingress route တစ်ခုကို create ဖို့လိုပါတယ်။ service တွေကို deployment နဲ့ ချိတ်ဖို့ဆိုရင် label တွေကို သုံးကြပါတယ်။ ingress မှာတော့ service နဲ့ ချိတ်ဖို့အတွက် service name နဲ့ service port  အသုံးပြုရပါမယ်။ ingress create မလုပ်ခင်မှာ အရေးကြီးဆုံးကတော့ ingress controller ရဲ့ loadbalancer ip ကို မိမိအသုံးပြုမယ်ံ domain မှာ dns record သတ်မှတ်ပေးရပါမယ်။ ဒါမှသာ user တွေက domain ကို access လုပ်တဲ့အခါ အဲ့ဒီ ingress controller ကိုရောက်သွားပါမယ်။ ingress controller ရဲ့ အလုပ်က user တွေခေါ်လိုက်တဲ့ domain ရဲ့ url ကို kubernetes ပေါ်မှာ ရှိတဲ့ ချိတ်ထားတဲ့  သက်ဆိုင်ရာ backend service တွေကို ပြန်ပြီး route လုပ်ပေးမှာဖြစ်ပါတယ်။ 
