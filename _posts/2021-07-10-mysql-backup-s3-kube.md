---
layout:     post
title:      "Backup Mysql To S3 On K8s"
subtitle:   "Backup MySql DB To S3 On Kubernetes Using CronJob"
date:       2021-07-10 4:00:00
author:     "Thaung Htike Oo"
header-img: img/black.jpg
catalog: true
tags:
    - Kubernetes 
    - AWS
---

<h2> How To Backup MySql Database To S3 On Kubernetes Using CronJon </h2>

ဒီနေ့မှာတော့ kubernetes ပေါ်မှာ statefulset နဲ့တင်ထားတဲ့ mysql db ကို cronjob ကိုသုံးပြီး aws s3 bucket ထဲကို backup လုပ်တာကို ရှင်းပြသွားမှာဖြစ်ပါတယ်။ Kubernetes ဆိုတာ containerized applications တွေအတွက် automate လုပ်ဖို့သုံးတာဖြစ်တယ်။ S3 ဆိုတာ က amazon က ထုတ်တဲ့ storage solution တစ်ခုဖြစ်တယ်။ 

<p>
cloud services တွေ popular မဖြစ်ခင်ကဆို ကျွန်တော်တို့ရဲ့ company တွေမှာ mysql ၊ mongodb ၊ postgresql အစရှိတဲ့ database တွေကို နေ့စဥ် crontab တွေ run ပြီး backup လုပ်ထားကြပါတယ်။ Backup ထားမှသာ တစ်ခုခု lose ဖြစ်သွားရင် restore ပြန်လုပ်ဖို့လွယ်ကူပါမယ်။ ဒီနေ့မှာဆိုရင်လည်း ကျွန်တော်က kubernetes ပေါ်မှာ mysql database ကို cronjob နဲ့ aws s3 ပေါ်မှာ backup လုပ်တာကိုရှင်းပြသွားပါမယ်။
</p>

<h2> Prequities </h2>
```bash
 Basic Kubernetes Knowledge 
 AWS Account
 MySql Basic
```
<h2> Deploy Mysql Statefulset On Kubernetes </h2>
<p>
ပထမဆုံးအနေနဲ့ mysql ကို statefulset တစ်ခုအနေနဲ့ kubernetes ပေါ်မှာ run ပေးရပါမယ်။ statefulset create မလုပ်ခင် statefulset အတွက်သုံးမယ့် volume ကိုစဥ်းစားရပါတော့မယ်။ volume provisioner တွေထဲမှာ on-prem အတွက်ဆို glusterfs ၊ nfs ၊ heketi ၊ openebs ၊ rook တို့ဟာလူသုံးများကြပါတယ်။ cloud kubernets services တွေဖြစ်တဲ့ AKS EKS GKE တို့အတွက်ဆို EBS ၊ AzureDisk ၊ GCE computeDisk တို့ကိုသုံးကြပါတယ်။
</p>
<p>
ဒီနေ့ မှာတော့ ကျွန်တော်က volume အကြောင်းကို ရေးမှာမဟုတ်တော့ အလွယ်တကူ သုံးနိုင်တဲ့ hostPath ကိုပဲ သုံးလိုက်ပါတယ်။
deployment အားလုံးကို default namespace မှာပဲ စမ်းမှာပါ။ အခု pv (persistent volume) တစ်ခုကို create လုပ်လိုက်ပါမယ်။ pv name ကိုတော့ mysql-pv လို့ပေးလိုက်ပါမယ်။ hostPath ဖြစ်လို့ class က manual ဖြစ်ပါတယ်။ RWO access mode ပေးထားပြီး 5GB ပေးသုံးထားပါတယ်။ PV နဲ့ PVC အကြောင်းကို volume အကြောင်းရေးတဲ့အခါ သေချာရေးပါဦီးမယ်။ ဒီ lab မှာ master တစ်ခုရယ် worker နှစ်ခုရယ် သုံးမှာဖြစ်ပါတယ်။ 
</p>

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```    

kubectl apply နဲ့ pv ကိုအောက်ကအတိုင်း create လိုက်ပါ။ ပြီးရင် kubectl get pv နဲ့ ကြည့်လိုက်ပါ။
```bash
kubectl apply -f pv.yaml
kubectl get pv
```

pv တစ်ခု ရပြီဆိုရင် mysql statefulset ကို စပြီး create နိုင်ပါပြီ။ statefulset အကြောင်းကိုလည်း အခုအနည်းငယ်ပဲရေးလိုက်ပါတယ်။ statefulset တစ်ခုအတွက် headless service တစ်ခုဆောက်ပေးရပါတယ်။ ကျွန်တော်ကတော့ clusterIP အနေနဲ့တန်းပြီး create လုပ်လိုက်ပါတယ်။ sts ရဲ့ pod replica တစ်ခုချင်းစီဟာကိုယ်ပိုင် pvc တစ်ခုလိုပါတယ်။ ဒါကြောင့် volumeTemplate ကိုတစ်ခါထဲထည့်ဆောက်ပေးရပါတယ်။ အောက်က yaml file ကနေ mysql sts တစ်ခုကိုဆောက်မှာပါ။ ဒါဆို mysql sts ကို create လိုက်ကြရအောင်။ mysql pod ရဲ့ pvc ဟာ ခုဏက pv နဲ့ သွား bound ပါလိမ့်မယ်။
```bash
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  ports:
  - port: 3306
    name: mysql
    targetPort: 3306
  selector:
    app: mysql
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql
        env:
          # Use secret in real usage
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: pvc-hostpath
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-pv
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi

```
kubectl create နဲ့ sts တစ်ခုဆောက်လိုက်ပါမယ်။
```bash
kubectl create -f mysql.yaml
```
create ပြီးသွားရင် kubectl get နဲ့ ကြည့်လိုက်ရင် mysql sts run နေတာကို အောက်ကလို တွေ့ရမှာဖြစ်ပါတယ်။
```bash
oot@master:~#  kubectl get all
NAME                                      READY   STATUS      RESTARTS   AGE
pod/mysql-0                               1/1     Running     0          2m

NAME            TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/mysql   ClusterIP     10.99.59.201     <pending>   3306/TCP          2m

NAME                     READY   AGE
statefulset.apps/mysql   1/1     2m
```
mysql service ကို external access ပေးချင်ရင် service ကို NodePort (သို့) LoadBalancer အဖြစ် expose ပေးဖို့လိုပါတယ်။ kubectl edit svc နဲ့ NodePort အဖြစ် expose လိုက်ပါမယ်။ ClusterIP နေရာမှာ NodePort နဲ့အစားထိုးလိုက်ပါ။
```bash
kubectl edit svc mysql
```
ပြီးသွားရင် kubectl get svc နဲ့ services တွေကို ပြန်စစ်လိုက်ပါ။ mysql service ဟာ nodePort အနေနဲ့ ရလာပါပြီ။ 
```bash
oot@master:~#  kubectl get svc

NAME            TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/mysql   NodePort      10.99.59.201     <pending>   3306:32237/TCP    54m
```
ဒီ lab မှာ ကျွန်တော့် master node ip ဟာ 10.0.0.4 ဖြစ်ပြီး worker node က 10.0.0.5 ဖြစ်ပါတယ်။ NodePort ဆိုတော့ mysql service ဟာ node တွေရဲ့ port 32237 မှာ access ရနေမှာဖြစ်ပါတယ်။ 10.0.0.4 master node မှာပဲ access လုပ်ကြည့်ရအောင်။
```bash
root@master:~# mysql -h 10.0.0.4 -u root --port 32237 -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 119
Server version: 8.0.25 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER 'root' IDENTIFIED WITH mysql_native_password BY 'Th@ungHt!ke00';

```
root password က statefulset yaml ထဲမှာထည့်ထားပါတယ်။ root password ပြန်ချိန်းပေးလိုက်ပါ။ sts yaml ထဲမှာ တစ်ခါထဲပြောင်းခဲ့လဲရပါတယ်။ ဒါဆိုရင်တော့ mysql အပိုင်းကပြီးပါပြီ။ backup လုပ်ဖို့အတွက် container တစ်ခု create ရပါမယ်။ ပြီးရင်တော့ s3 ထဲမှာ bucket တစ်ခုဆောက်ရပါမယ်။'mysql-cron-test' ဆိုပြီး ကျွန်တော်ကတော့ bucket တစ်ခု ဆောက်ထားပါတယ်။ bucket ထဲမှာ mysql dump တွေကိုသိမ်းဖို့ folder တစ်ခု create ခဲ့ပါ။ အခု backup လုပ်ပေးမယ့် container တစ်ခုကို create ပါမယ်။
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-database-backup
type: Opaque
data:
  database_password: VGhAdW5nSHQha2UwMAo=
  slack_webhook_url: cGFzc3dvcmQK
---
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: my-database-backup
spec:
  schedule: "0 22 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: my-database-backup
            image: ghcr.io/benjamin-maynard/kubernetes-cloud-mysql-backup:v2.5.0
            imagePullPolicy: Always
            env:
              - name: AWS_ACCESS_KEY_ID
                value: "<your_aws_access_key>"
              - name: AWS_SECRET_ACCESS_KEY
                value: "<your_aws_secret_key>"
              - name: AWS_DEFAULT_REGION
                value: "us-east-1"
              - name: AWS_BUCKET_NAME
                value: "mysql-cron-test"
              - name: AWS_BUCKET_BACKUP_PATH
                value: "/dump"
              - name: TARGET_DATABASE_HOST
                value: "10.0.0.4"
              - name: TARGET_DATABASE_PORT
                value: "32237"
              - name: TARGET_DATABASE_NAMES
                value: "mysql"
              - name: TARGET_DATABASE_USER
                value: "root"
              - name: TARGET_DATABASE_PASSWORD
                valueFrom:
                   secretKeyRef:
                     name: my-database-backup
                     key: database_password
              - name: BACKUP_TIMESTAMP
                value: "_%Y_%m_%d"              
              - name: SLACK_ENABLED
                value: "false"
              - name: SLACK_CHANNEL
                value: "#chatops"
              - name: SLACK_WEBHOOK_URL
                valueFrom:
                   secretKeyRef:
                     name: my-database-backup
                     key: slack_webhook_url
          restartPolicy: Never
```
yaml file ထဲမှာ လိုအပ်တာတစ်ခုချင်းထည့်ပေးရပါမယ်။ database_password က Th@ungHtikeOo ကို base64 နဲ့ encode လုပ်ထားတာဖြစ်ပါတယ်။ aws access key နဲ့ secret key ကိုလည်းမိမိတို့ရဲ့ key တွေကိုထည့်ပေးပါ။
bucket region က bucket create ခဲ့တဲ့ region ၊ db host က k8s node ip တစ်ခုထည့်ရမယ်၊ db port က node port ဖြစ်တဲ့ 32237 ၊ target db name က backup လုပ်ချင်တဲ့ db name ၊ slack ကို disabled ခဲ့တယ်။ 

production ဆို slack channel နဲ့ connect လုပ်ပေးလိုက်ပါ။ cron ရဲ့ value အရ နေ့တိုင်း ည (၁၀) နာရီမှာ database ကို backup မှာဖြစ်ပါတယ်။ အားလုံးသတ်မှတ်ပြီးသွားရင် job ကို create လုပ်လိုက်ပါတော့မယ်။

```bash
kubectl create -f mysql_cronjob.yaml
```
ပြီးသွားရင်တော့ ည (၁၀) နာရီရောက်တဲ့အခါကျွန်တော်တို့ရဲ့ db ကို စပြီး backup မယ့် job တစ်ခု run နေတာကိုတွေ့ရပါလိမ့်မယ်။ backup ပြီးတဲ့အခါ job က completed ဖြစ်ပြီး terminated သွားပါမယ်။ နောက်ရက် ည (၁၀)နာရီ ရောက်တဲ့အခါ နောက် job တစ်ခုကို run နေပါလိမ့်မယ်။
```bash
root@master:~#  kubectl get all
NAME                                      READY   STATUS      RESTARTS   AGE
pod/my-database-backup-1625918040-678nz   0/1     Completed   0          30s
pod/mysql-0                               1/1     Running     0          54m

NAME            TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/mysql   LoadBalancer   10.99.59.201   <pending>     3306:32237/TCP   54m

NAME                     READY   AGE
statefulset.apps/mysql   1/1     54m

NAME                                      COMPLETIONS   DURATION   AGE
job.batch/my-database-backup-1625918040   1/1           11s        30s

NAME                               SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/my-database-backup   * * * * *   False     0        32s             38s
```
aws s3 ထဲမှာလည်း အောက်ကပုံအတိုင်း sql dump တစ်ခုကို တွေ့ရပါလိမ့်မယ်။ 
![s3](https://raw.githubusercontent.com/thaunggyee/thaunggyee.github.io/master/img/s3.png)

ဒါဆိုရင်တော့ ဒီနေ့lab က ဒီလောက်ပါပဲ။ ဒီ lab လေးကို လုပ်ကြည့်ကြဖို့လည်း တိုက်တွန်းပါတယ်။ အားလုံးကိုကျေးဇူးတင်ပါတယ်။ 

Thanks for reading ...

