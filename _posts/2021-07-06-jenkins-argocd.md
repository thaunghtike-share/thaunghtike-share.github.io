---
layout:     post
title:      "Gitops In Kubernetes With Jenkins And ArgoCD"
subtitle:   "Gitops At Scale With Jenkins And ArgoCD On Kubernetes"
date:       2021-07-19 18:00:00
author:     "Thaung Htike Oo"
header-img: img/black.jpg
catalog: true
tags:
    - Kubernetes
    - Jenkins
    - CICD
---    

<h2> Introduction </h2>

ကျွန်တော်ဒီနေ့တော့ GitOps အကြောင်းကိုနားလည်သလောက် ရှင်းပြသွားမှာဖြစ်ပါတယ်။ ပြီးရင် demo တစ်ခုနဲ့စမ်းပြပါမယ်။ ဒီနေ့အတွက်ခေါင်းစဥ် ကိုတော့ 'gitops in kubernetes with jenkins and argocd' လို့ပေးထားပါတယ်။ ဒီနေ့ demo မှာ jenkins ကို docker image တွေ build လုပ်ဖို့၊ pushလုပ်ဖို့ စသည်တို့အတွက် CI tool အဖြစ်သုံးမှာဖြစ်ပြီး argocd ကိုတော့ kubernetes အတွက် continuous deployments တွေလုပ်ဖို့သုံးမှာဖြစ်ပါတယ်။
Jenkins install လုပ်နည်းကို ရှင်းပြပြီးပြီ ဆိုတော့ ပထမဆုံးအနေနဲ့ jenkins pipeline တစ်ခုကို create ပါမယ်။ jenkins install လုပ်ပုံကို အောက်က link မှာဖတ်နိုင်ပါတယ်။

[How To Install Jenkins On Ubuntu 20.04 LTS](https://thaunggye.github.io/2021-07-07-jenkins)

<h2> Prerequities </h2>

<ul>
    <li> Jenkins Server </li>
    <li> Kubernetes Cluster </li>
</ul>    

<h2> Create a Pipeline </h2>

Jenkins dashboard ထဲရောက်သွားပြီဆိုရင် New Item ကနေ pipeline project တစ်ခု create ပါမယ်။ name ကိုတော့ django_app လို့ပေးထားပါတယ်။ docker နဲ့ဆိုင်တာတွေကို jenkins မှာလုပ်မှာဆိုတော့ Manage Plugins ထဲကနေ docker plugins ကို install ပေးရပါမယ်။ 

![dapp](https://raw.githubusercontent.com/thaunggye/thaunggye.github.io/master/img/dapp.png)

github project မှာ demo အတွက်သုံးမယ်ံ repo ကိုထည့်ပါမယ်။

![p1](https://raw.githubusercontent.com/thaunggye/thaunggye.github.io/master/img/dp1.png)

pipeline script ထဲမှာတော့ docker image ကို build လုပ်ဖို့၊ push လုပ်ဖို့အတွက်ရေးပေးမှာဖြစ်ပါတယ်။ script ကို အောက်မှာကြည့်နိုင်ပါတယ်။ ပထမအဆင့်မှာ git repo နဲ့ branch ကိုထည့်မယ်၊ ပြီးရင် docker image ကို build မယ် name က mcapp ၊ tag က 1.$BUILD_NUMBER ဆိုပြီးပေးမယ်။ ပြီးရင် docker login လုပ်ပြီး docker hub ထဲကို push လိုက်ပါမယ်။

```yaml
pipeline {
    agent any
    
    environment {
        DOCKER_USER = 'tho861998'
        DOCKER_REGISTRY = 'docker.io'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/thaunggyee/django_mc_app.git'
            }
        }
        stage('docker image building') {
            steps {
                sh 'docker build -t tho861998/mcapp:1.${BUILD_NUMBER} .'
            }
        }
        stage('docker login') {
            steps {
                sh 'docker login -u $DOCKER_USER -p $DOCKER_PASSWORD $DOCKER_REGISTRY'    
            }
        }
        stage('docker push') {
            steps {
                sh 'docker push tho861998/mcapp:1.${BUILD_NUMBER}'
            }
        }
    }    
}
```
ပြီးသွားရင်တော့ save and apply ကိုနှိပ်လိုက်ပါ။ ဒါဆိုရင် pipeline ကို build လုပ်လို့ရပါပြီ။ ဘေးက BUILD NOW ဆိုတာကိုနှိပ်လိုက်ရင် pipeline တစ်ခု build နေတာကိုအခုလိုတွေ့ရမှာဖြစ်ပါတယ်။

![finish](https://raw.githubusercontent.com/thaunggye/thaunggye.github.io/master/img/fs.png)

build တာပြီးသွားပြီဆိုရင် docker hub ထဲမှာ mcapp ဆိုပြီး image တစ်ခုတွေ့ရမှာဖြစ်ပါတယ်။ tag ကတော့ build တစ်ခုပဲရှိသေးတော့ 1.1 တစ်ခုပဲရှိပါဦးမယ်။

![mcapp1](https://raw.githubusercontent.com/thaunggye/thaunggye.github.io/master/img/mcapp1.png)

<h2> Setup k8s cluster with Kind </h2>

Demo အတွက် kubernetes clusterတစ်ခုလိုပါတယ်။ ကျွန်တော်ကတော့ kind ကိုသုံးပြီး cluster တစ်ခု create လိုက်ပါမယ်။ go နဲ့ docker install အရင် install ရပါမယ်။ ပြီးတဲ့အခါ kind ကို install လုပ်ပေးရပါမယ်။

```bash
GO111MODULE="on" go get sigs.k8s.io/kind@v0.11.1
```
kind install ပြီးရင်တော့ cluster တစ်ခု create လိုက်ပါမယ်။

```bash
thaunghtikeoo@thaunghtikeoo:~$ kind create cluster 
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Thanks for using kind! 😊
```

ဒါဆိုရင် kubernetes တစ်ခု ready ဖြစ်ပါပြီ။ kubernetes cluster ပေါ်မှာ continuous deployment ကိုစမ်းဖို့အတွက် argo-cd ကိုအရင် install ပါမယ်။ argocd install ဖို့အတွက် အောက်က command တွေကိုရိုက်ထည့်ပေးပါ။

```bash
$ kubectl create ns argocd
$ helm repo add argo https://argoproj.github.io/argo-helm
$ helm install argocd argo/argo-cd -f https://gist.githubusercontent.com/pcrete/250896d4ff90ce2afa496c9f515b6be5/raw/0057742a532e8dfd578c139385537cd39de07f03/argocd-values.yaml --version 2.6.0 --namespace argocd
```
argocd server ကို browser ကခေါ်ဖို့အတွက် argocd-server service ရဲ့ ip ကို ကြည့်ရပါမယ်။ 

```bash
thaunghtikeoo@thaunghtikeoo:~$ kubectl get svc -n argocd
NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
argocd-application-controller   ClusterIP   10.96.122.24    <none>        8082/TCP            6m4s
argocd-dex-server               ClusterIP   10.96.32.12     <none>        5556/TCP,5557/TCP   6m4s
argocd-redis                    ClusterIP   10.96.196.234   <none>        6379/TCP            6m4s
argocd-repo-server              ClusterIP   10.96.205.112   <none>        8081/TCP            6m4s
argocd-server                   ClusterIP   10.96.215.91    <none>        80/TCP,443/TCP      6m4s
```
ဒါဆိုရင် 10.96.215.91 ကို browser ကနေခေါ်လိုက်ပါ။ default username က admin ဖြစ်ပြီး password က argocd-server ရဲ့ pod name 'argocd-server-5747c8dc9f-bnvcv' ဖြစ်ပါတယ်။ login ပြီးတဲ့အခါ console ကိုအောက်ကလိုတွေ့ရမှာဖြစ်ပါတယ်။

![acon](https://raw.githubusercontent.com/thaunggye/thaunggye.github.io/master/img/acon.png)

ပုံမှန်ဆိုရင် kubectl command နဲ့ kubernetes ပေါ်မှာ deploy လုပ်ကြပါတယ်။ argo မှာကျ k8s ပေါ်တင်မယ့် resource တွေကို သေချာ plan ချပြီးသွားတဲ့အခါ main branch ပေါ်တင်ကြပါတယ်။ argocd က kubernetes နဲ့ ဆိုင်တဲ့ deployment ၊ service ၊ ingress စတာတွေကို git repo ကနေယူပြီး k8s ပေါ် deploy လုပ်ပေးတာပါ။ commit တစ်ခုဖြစ်တိုင်း git repo နဲ့ k8s ကို sync လုပ်ပေးတာပါ။ အသေးစိတ်ကိုတော့ argocd ဆိုပြီး ရှာဖတ်နိင်ပါတယ်။ ဒါဆိုရင် အခု argocd ပေါ်မှာ jenkins နဲ့ build ခဲ့တဲ့ image ကိုသုံးပြီး app တစ်ခု create ပြီး k8s ပေါ်တင်ပါမယ်။ 

kubernetes deployment file ကို အပေါ်မှာသုံးခဲ့တဲ့ git repo ထဲမှာ kubernetes ဆိုတဲ့ folder အောက်မှာထည့်ပါမယ်။ mcapp deployment နဲ့ service အတွက် အောက်က yaml file ကို သုံးပါမယ်။

```yaml
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mcapp
  labels:
    app: mcapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mcapp
  template:
    metadata:
      labels:
        app: mcapp
    spec:
      containers:
      - name: mcapp
        image: tho861998/mcapp:1.1
        ports:
        - containerPort: 8080
 
---
apiVersion: v1
kind: Service
metadata:
  name: mcapp
spec:
  selector:
    app: mcapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

yaml file ရေးပြီးပြီဆိုရင် argocd app တစ်ခု create ပါမယ်။ New App ကနေ app name-mcapp ၊ project-default ၊ sync policy မှာတော့ auto ၊ repo မှာ github က repo ၊ path က kubernetes ၊ cluster က https://kubernetes.default.svc ၊ namespace မှာ default အကုန်ပေးပြီးရင် create လိုက်ပါ။ ဒါဆိုရင် အောက်မှာတွေ့ရတဲ့အတိုင်း mcapp ဆိုပြီး application တစ်ခုရလာပါပြီ။

![am](https://raw.githubusercontent.com/thaunggye/thaunggye.github.io/master/img/am.png)

auto sync ပေးခဲ့တော့ sync လုပ်ပြီးတဲ့အခါ app ကို click တစ်ချက်နှိပ်လိုက်ရင် deploy နဲ့ svc run နေတာကိုတွေ့ရမှာပါ။

![am1](https://raw.githubusercontent.com/thaunggye/thaunggye.github.io/master/img/am1.png)

k8s terminal ကကြည့်ရင်လည်း deployment နဲ့ service run နေတာကိုတွေ့ရမှာဖြစ်ပါတယ်။

```bash
thaunghtikeoo@thaunghtikeoo:~$ kubectl get all
NAME                        READY     STATUS    RESTARTS   AGE
pod/mcapp-d949c5bc7-srf86   1/1       Running   0          5m22s

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP   158m
service/mcapp        ClusterIP   10.96.9.118   <none>        80/TCP    5m22s

NAME                    READY     UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mcapp   1/1       1            1           5m22s

NAME                              DESIRED   CURRENT   READY     AGE
replicaset.apps/mcapp-d949c5bc7   1         1         1         5m22s
```
mcapp pod ကို port-forward လုပ်ကြည့်ရင် monthly challenges app ကိုတွေ့ရမှာဖြစ်ပါတယ်။ 

![mcw](https://raw.githubusercontent.com/thaunggye/thaunggye.github.io/master/img/mcw.png)

အာလုံးကိုကျေးဇူးတင်ပါတယ်ခင်ဗျာ။ ကျန်းမာရေးကောင်းအောင်လည်းဂရုစိုက်ကြပါ။ 

Thanks for reading ...





