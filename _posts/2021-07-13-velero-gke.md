---
layout:     post
title:      "Backup & Restore Objects In GKE"
subtitle:   "How To Bakup And Restore Objects In GKE Cluster"
date:       2021-07-13 4:00:00
author:     "Thaung Htike Oo"
header-img: img/black.jpg
catalog: true
tags:
    - Kubernetes
    - GKE
---

<h2> How To Backup And Restore Objects In GKE Cluster </h2>

Kubernetes Cluster တွေပေါ်မှာ deploy လုပ်ထားတဲ့ workload တွေနဲ့ တစ်ခြားသော resources တွေကို backup လုပ်ဖို့ဆို velero ဟာ အကောင်းဆုံး tool တစ်ခုဖြစ်တယ်ဆိုတာ အားလုံးလက်ခံကြမှာပါ။ on-prem မှာရှိတဲ့ 
kubernetes cluster တွေကို minio နဲ့ velero ကိုတွဲသုံးပြီး backup and restore လုပ်ကြပါတယ်။ ကျွန်တော် ဒီနေ့မှာတော့ GKE cluster တစ်ခုပေါ်မှာ deploy ထားတဲ့ workloads တွေကို velero သုံးပြီးဘယ်လို 
backup and restore လုပ်မလဲဆိုတာကိုရေးမှာဖြစ်ပါတယ်။

<h2> Prerequities </h2>

```yaml
google clod console
gke cluster
kubernetes basic
````
<h2> Why Velero? </h2>

Kubernetes Cluster တွေကို backup ဖို့ tools တွေအများကြီးထဲက မှ velero ကိုဘာလို့ ရွေးချယ်ရလဲဆိုရင် velero က opensource ဖြစ်ပြီး on-prem cluster တွေအတွက်ကော cloud cluster တွေအတွက်ကော support
လုပ်ပါတယ်။ အသေးစိတ်ကိုတော့ velero ရဲ့ official doc မှာဖတ်နိုင်ပါတယ်။ ဒါဆိုရင်အခုပဲ lab ကိုစလိုက်ရအောင်နော်။ 

GCP console ထဲကနေ Kubernetes Engine ဆိုပြီးရှာလိုက်ပါ။ ပြီးရင် node (၃)ခုနဲ့ cluster တစ်ခုကိုဆောက်ပေးပါ။ cluster create ပြီးရင်တော့ အောက်ကလိုတွေ့ရမှာပဲဖြစ်ပါတယ်။

![gke](https://raw.githubusercontent.com/thaunggyee/thaunggyee.github.io/master/img/gke.png)

ပြီးသွားရင် ဘေးက အစက် (၃)စက် ကနေ connect ကိုနှိပ်လိုက်ပါ။ RUN IN CLOUD SHELL ကို click လိုက်ရင် cloud shell ထဲကို ရောက်သွားပါပြီ။ Enter ခေါက်ပြီး Authorize လုပ်ပေးလိုက်ပါ။ အားလုံးပြီးရင်တော့ kubectl နဲ့ gke cluster ကို စတင်အသုံးပြုလို့ရပါပြီ။

![connect](https://raw.githubusercontent.com/thaunggyee/thaunggyee.github.io/master/img/connect.png)

kubectl get nodes နဲ့အရင် nodes တွေကိုစစ်ကြည့်လိုက်ပါ။ ready ဖြစ်နေရင် deployment တွေစလုပ်လို့ရပါပြီ။

```bash
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ kubectl get nodes
NAME                                        STATUS   ROLES    AGE   VERSION
gke-gcp-velero-default-pool-5b41b772-97v2   Ready    <none>   27m   v1.19.9-gke.1900
gke-gcp-velero-default-pool-5b41b772-ckhm   Ready    <none>   27m   v1.19.9-gke.1900
gke-gcp-velero-default-pool-5b41b772-hm2s   Ready    <none>   27m   v1.19.9-gke.1900
```
backup နဲ့ restore အတွက် tar file တွေကိုသိမ်းဖို့ cloud storage bucket တစ်ခုကို အောက်ပါအတိုင်း create နိုင်ပါတယ်။ 

```bash
BUCKET=tho-velero-gcs
gsutil mb gs://$BUCKET/
```
create ပြီးသွားတဲ့အခါ gsutil ls နဲ့ ကြည့်လိုက်ရင် အပေါ်က create ခဲ့တဲ့ bucket တစ်ခုရောက်နေတာကို တွေ့ရမှာပါ။

```bash
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ gsutil ls
gs://tho-velero-gcs/
```
နောက်တစ်ဆင့်ကတော့ gke cluster ပေါ်မှာ deploy ထားတဲ့ workloads တွေ pvc တွေဟာ compute disk တွေထဲမှာ store လုပ်ထားတာပါ။ ဒါကြောင့် backup file တွေသိမ်းမယ့် bucket အတွက် gke ပေါ်က resources တွေကို backup and restore လုပ်ဖို့ဆို compute disk တွေနဲ့ ဆိုင်တဲ့ permission တွေသတ်မှတ်ပေးဖို့လိုပါတယ်။ 

project id ကိုသိဖို့ current config setting ကိုကြည့်ပါ။

```bash
gcloud config list
```
ရလာတဲ့ project id value ကို PROJECT_ID ဆိုတဲ့ variable တစ်ခုသတ်မှတ်လိုက်ပါ။

```bash
PROJECT_ID=$(gcloud config get-value project)
```
bucket ကို compute disk နဲ့ ဆိုင်တဲ့ permission တွေက တိုက်ရိုက် assign ချပေးလို့မရပါဘူး။ service account တွေကတစ်ဆင့် လုပ်ပေးမှာပါ။ ဒါကြောင့် service account တစ်ခုကို အရင် create ပေးရပါမယ်။

```bash
gcloud iam service-accounts create velero \
    --display-name "Velero service account"
```
ပြီးရင် service account email ကို variable သတ်မှတ်ပေးရပါမယ်။

```bash
SERVICE_ACCOUNT_EMAIL=$(gcloud iam service-accounts list \
  --filter="displayName:Velero service account" \
  --format 'value(email)')
```
velero service account ဟာ compute disk တွေနဲ့ bucket ကို ချိတ်ဆက်ပေးမှာပါ။ ဒါကြောင့် velero sa ကို လိုအပ်တဲ့ permissions တွေ roles တွေသတ်မှတ်ပေးရပါမယ်။

```yaml
ROLE_PERMISSIONS=(
    compute.disks.get
    compute.disks.create
    compute.disks.createSnapshot
    compute.snapshots.get
    compute.snapshots.create
    compute.snapshots.useReadOnly
    compute.snapshots.delete
    compute.zones.get
)
```
permission တွေကို velero server ဆိုတဲ့ role ထဲကို ထည့်ပါ။ ပြီးရင်တော့ အဲ့ role ကို velero service account ထဲမှာ အောက်ကအတိုင်း bind လိုက်ပါ။

```bash
gcloud iam roles create velero.server \
    --project $PROJECT_ID \
    --title "Velero Server" \
    --permissions "$(IFS=","; echo "${ROLE_PERMISSIONS[*]}")"

gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member serviceAccount:$SERVICE_ACCOUNT_EMAIL \
    --role projects/$PROJECT_ID/roles/velero.server
```
အကုန်ပြီးသွားရင် service account နဲ့ bucket နဲ့ချိတ်ပေးလိုက်ပါ။ ဒါဆိုရင်တော့ service account တစ်ခုနဲ့ permission သတ်မှတ်တာပြီးပါပြီ။

```yaml
gsutil iam ch serviceAccount:$SERVICE_ACCOUNT_EMAIL:objectAdmin gs://${BUCKET}
```
velero ကို gke ပေါ်မှာ install ဖို့အတွက်ဆို ခုဏက create ခဲ့တဲ့ service account ရဲ့ key တွေကိုလိုအပ်ပါတယ်။ အောက်ပါ command နဲ့ထုတ်လိုက်ရင် cloud shell ရဲ့ current directory မှာ credentials-velero ဆိုတဲံ့ key file တစ်ခုရလာပါလိမ့်မယ်။

```yaml
gcloud iam service-accounts keys create credentials-velero \
    --iam-account $SERVICE_ACCOUNT_EMAIL
```
<h2> Install Velero </h2>

velero cli ကို install လုပ်ပေးရပါမယ်။ 

```bash
wget https://github.com/vmware-tanzu/velero/releases/download/v1.6.1/velero-v1.6.1-linux-amd64.tar.gz
tar xzvf velero-*
sudo mv velero-v1.6.1-linux-amd64/velero /usr/local/bin/velero
sudo chmod +x /usr/local/bin/velero
velero version
````
<h2> Install Velero On GKE </h2>

velero cli ကို install ပြီးတဲ့နောက်မှာ velero ကို cluster ပေါ်မှာ deploy လုပ်ပေးဖို့လိုပါတယ်။

```bash
velero install \
    --provider gcp \
    --plugins velero/velero-plugin-for-gcp:v1.2.0 \
    --bucket $BUCKET \
    --secret-file ./credentials-velero
```
install ပြီးသွားတဲ့အခါ အောက်ကလိုတွေ့ရပါလိမ့်မယ်။ 

```bash
ServiceAccount/velero: attempting to create resource
ServiceAccount/velero: attempting to create resource client
ServiceAccount/velero: created
Secret/cloud-credentials: attempting to create resource
Secret/cloud-credentials: attempting to create resource client
Secret/cloud-credentials: created
BackupStorageLocation/default: attempting to create resource
BackupStorageLocation/default: attempting to create resource client
BackupStorageLocation/default: created
VolumeSnapshotLocation/default: attempting to create resource
VolumeSnapshotLocation/default: attempting to create resource client
VolumeSnapshotLocation/default: created
Deployment/velero: attempting to create resource
Deployment/velero: attempting to create resource client
Deployment/velero: created
Velero is installed! ⛵ Use 'kubectl logs deployment/velero -n velero' to view the status.
```
velero က gke ပေါ်မှာ run နေပြီဆိုတာကို deployment ကို ကြည့်ပြီးသိနိုင်ပါတယ်။

```yaml
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ kubectl get all -n velero
NAME                          READY   STATUS    RESTARTS   AGE
pod/velero-664d8dc67f-cgtds   1/1     Running   0          2m17s

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/velero   1/1     1            1           2m18s
NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/velero-664d8dc67f   1         1         1       2m18s
```
အားလုံးပြီးသွားရင်တော့ backup & restore ကို စမ်းဖို့အတွက် nginx deployment တစ်ခုကို create လိုက်ပါမယ်။

```yaml
kubectl create deploy nginx --image nginx
```
ဒါဆိုရင် defautl namespace မှာ nginx deployment တစ်ခု run နေတာကိုတွေ့ရပါလိမ့်မယ်။ 

```yaml
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6799fc88d8-hl57c   1/1     Running   0          12s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.40.0.1    <none>        443/TCP   76m
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           13s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6799fc88d8   1         1         1       13s
```
ဒါဆိုရင်အခုပဲ backup လုပ်တာကို စမ်းကြည့်ပါတော့မယ်။ backup လုပ်တဲ့အခါ option တွေများစွာရှိပြီး manually backup လုပ်နိုင်သလို cron တွေနဲ့လည်း backup လုပ်နိုင်ပါတယ်။ အောက်က example ကတော့ defautl namespace တစ်ခုလုံးကို backup လုပ်လိုက်တာပါ။

```bash
velero create backup firstbackup --include-namespaces default
```
backup create ပြီးရင်တော့ velero backet get နဲ့ကြည့်လိုက်ပါ။ complete ဖြစ်နေတာကိုတွေ့ရမှာဖြစ်ပါတယ်။

```
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ velero backup get
NAME          STATUS      ERRORS   WARNINGS   CREATED                         EXPIRES   STORAGE LOCATION   SELECTOR
firstbackup   Completed   0        0          2021-07-13 07:35:46 +0000 UTC   29d       default            <none>
```
ကျွန်တော်တို့ cloud storage ထဲက bucket မှာလည်း firstbackup ဆိုပြီး အောက်ပါအတိုင်းတွေ့ရမှာဖြစ်ပါတယ်။

![bucket](https://raw.githubusercontent.com/thaunggyee/thaunggyee.github.io/master/img/backup.png)

backup တစ်ခုချင်းစီိကို detail ကြည့်ချင်ရင်တော့ အောက်က command တွေနဲ့ ကြည့်နိုင်ပါတယ်။

```yaml
velero backup get
velero backup describe <name>
velero backup logs <name>
kubectl get backup -n velero
kubectl describe backup <name> -n velero
````
backup စမ်းတာပြီးပြီဆိုတော့ restore အပိုင်းကိုစမ်းပါမယ်။ ခုဏက create ခဲ့တဲ့ backup ကိုပဲ restore ပါမယ်။ defautl namespace မှာရှိတဲ့ nginx deployment ကို delete လိုက်ပါမယ်။

```yaml
kubectl delete all --all --force
```
ဒါဆိုရင်ကျွန်တော်ရဲ့ default namespace မှာဘာမှမရှိတော့ပါဘူး။ kubectl get all  နဲ့ကြည့်လိုက်ပါ။ 

```bash
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.40.0.1    <none>        443/TCP   82s
```
default namespace မှာ backup ခဲ့တာတွေကို velero restore နဲ့ စမ်းကြည့်ပါမယ်။

```bash
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ velero restore create firstrestore --from-backup firstbackup
Restore request "firstrestore" submitted successfully.
Run `velero restore describe firstrestore` or `velero restore logs firstrestore` for more details.
```
restore ပြီးတဲ့အခါ default namespace မှာ backup လုပ်ခဲ့တဲ့ nginx deployment run နေတာကိုတွေ့ရမှာဖြစ်ပါတယ်။

```bash
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6799fc88d8-hl57c   1/1     Running   0          3m16s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.40.0.1    <none>        443/TCP   6m37s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           3m17s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6799fc88d8   1         1         1       3m17s
```
velero restore get နဲ့ စစ်ကြည့်ရင်လည်း complete ဖြစ်နေတာကိုတွေ့ရမှာဖြစ်ပါတယ်။ 

```bash
thaunghtikeoo_tho1234@cloudshell:~ (clever-circlet-317904)$ velero restore get
NAME           BACKUP        STATUS      STARTED                         COMPLETED                       ERRORS   WARNINGS   CREATED                         SELECTOR
firstrestore   firstbackup   Completed   2021-07-13 07:49:20 +0000 UTC   2021-07-13 07:49:22 +0000 UTC   0        1          2021-07-13 07:49:20 +0000 UTC   <none>
```
console ပေါ်က bucket မှာလည်း restores ဆိုပြီး directory တစ်ခုကိုတွေ့ရမှာဖြစ်ပါတယ်။

![restor](https://raw.githubusercontent.com/thaunggyee/thaunggyee.github.io/master/img/restore.png)

အခြားသော backup option တွေကို အသေးစိတ်မရှင်းပြတော့ပါဘူး ။ exclude-namespaces ၊ exclude-resource ဆိုပြီး စသည်ဖြင့်လည်း backup လုပ်နိုင်ပါသေးတယ်။

```yaml
velero backup create nginxbackup --include-namespaces default --exclude-resources pods
velero backup create name --exclude-namespaces kube-system
velero backup create name --exclude-resources secrets/configmaps
```
cronjob တွေကိုသုံးပြီး backup ချင်ရင်တော့ အောက်ပါအတိုင်း backkup နိုင်ပါတယ်။ 

```yaml
kubectl schedule create firstbackup --scheudle="0 22 * * *" --include-namespace default  --exclude-resources secrets/configmaps
```

<h2> Conclusion </h2>

ကျွန်တော်ရှင်းပြခဲ့တာတွေကို နားလည်သဘောပေါက်လိမ့်မယ်လို့ထင်ပါတယ်။ နည်းနည်း ရှည်သွားပြီဆိုတော့ ဒီမှာပဲစာရေးတာကိုအဆုံးသတ်လိုက်ပါတော့မယ်။ PART-II မှာ backup storage location အကြောင်းကို ဆက်လက်ဆွေးနွေးသွားပါမယ်။
အားလုံးပဲ ဒီကပ်ရောဂါဆိုးကြီးကနေ အမြန် ကျော်ဖြတ်နိုင်ပါစေလို့ဆုတောင်းရင်း အဆုံသတ်လိုက်ပါတယ်။

Thanks for reading ...

