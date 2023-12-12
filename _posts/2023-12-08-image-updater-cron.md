---
layout:     post
title:   "Argocd Image Updater Part-II"
date:       2023-06-08 15:13:18 +0200
image: argocdimageupdater.png
tags:
    - devops
    - argocd
categories: DevOps
---

<p>EKS ပေါ်မှာ  argocd image updater အသုံးပြုလို့ရပြီလို့ထင်ခဲ့ပေမယ့် ပြသနာတစ်ခုရှိလာပါတယ်။ azure ရဲ့ AKS cluster ပေါ်မှာတော့ အဆင်ပြေပေမယ့် eks မှာတော့ docker registry secret ဟာ ၆နာရီတိုင်း refresh ပြန်လုပ်နေတာမလို့ ကျွန်တော်တို့ရဲ့ ECR နဲ့ argocd image updater တို့အကြားမှာ authentiacate လုပ်ဖို့အခက်အခဲရှိသွားပါတယ်။</p>

<p>သေချာရှင်းပြရရင်တော့ အောက်ပါအတိုင်း docker secret တည်ဆောက်စဥ်က aws ecr get-login-password ကနေ ရတဲ့ token ဟာ security အရပြန်ပီး refresh လုပ်ပေးဖို့လိုပါတယ်။ </p>

```
kubectl create secret docker-registry mycontainerregistry-secret \
  --docker-server=acc_id.dkr.ecr.ap-southeast-1.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region ap-southeast-1) \
  --namespace=argocd
```
<p>ဒါကတော့ ကျွန်တော့်ရဲဲ့ image updater နဲ့ချိတ်ဖို့ configmap ဖြစ်ပါတယ်။ </p>

```
apiVersion: v1
data:
  log.level: debug
  registries.conf: |
    registries:
    - name: AWS ECR
      prefix: accound_id.dkr.ecr.ap-southeast-1.amazonaws.com
      api_url: https://account_id.dkr.ecr.ap-southeast-1.amazonaws.com
      credentials: pullsecret:argocd/mycontainerregistry-secret
      ping: yes
      default: true
      insecure: no
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/name: argocd-image-updater-config
    app.kubernetes.io/part-of: argocd-image-updater
  name: argocd-image-updater-config
  namespace: argocd
```

<p>တစ်ခြားနည်းလမ်းတွေလည်းရှိကောင်းရှိနိုင်ပါသေးတယ်။ ဒီနည်းလမ်းကတော့ကျွန်တော့်အတွက်ပိုပြီး လွယ်ကူသလို အဆင်လည်းအခုထိပြေနေတဲ့အတွက် လိုအပ်တဲ့သူများယူနိုင်ပါတယ်။ အောက်ပါအတိုင်း cronjob တစ်ခုကို run ပေးလိုက်ရင် token expire ဖြစ်တာကိုကာကွယ်နိုင်ပီး image updater ကိုလည်း
ecr နဲ့  authenticate သေချာလုပ်နိုင်ပါပြီ။</p>

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: argocd-image-updater-ecr-token-refresh-sa
  namespace: argocd
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: argocd-image-updater-ecr-token-refresh
rules:
- apiGroups: [""]
  resources: ["secrets","pods","configmaps"]
  verbs: ["get", "delete", "list", "watch", "create"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "delete", "list", "watch", "create", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: argocd-image-updater-ecr-token-refresh-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argocd-image-updater-ecr-token-refresh
subjects:
- kind: ServiceAccount
  name: argocd-image-updater-ecr-token-refresh-sa
  namespace: argocd
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: argocd-image-updater-ecr-token-refresh
  namespace: argocd
spec:
  schedule: "0 */6 * * *" # Schedule to run every 1 hour
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: argocd-image-updater-ecr-token-refresh-sa
          containers:
          - name: kubectl
            image: acc_id.dkr.ecr.ap-southeast-1.amazonaws.com/kubectl-awscli:latest
            command:
              - "/bin/sh"
              - "-c"
            args:
              - |
                export AWS_ACCESS_KEY_ID=********
                export AWS_SECRET_ACCESS_KEY=********
                export AWS_DEFAULT_REGION=ap-southeast-1

                kubectl delete secret mycontainerregistry-secret -n argocd
                ecr_token=$(aws ecr get-login-password --region ap-southeast-1)

                kubectl create secret docker-registry mycontainerregistry-secret \
                --docker-server=429726127752.dkr.ecr.ap-southeast-1.amazonaws.com \
                --docker-username=AWS \
                --docker-password=$ecr_token \
                --namespace=argocd

                kubectl delete cm argocd-image-updater-config -n argocd
                kubectl apply -f - <<EOF
                apiVersion: v1
                data:
                  log.level: debug
                  registries.conf: |
                    registries:
                    - name: AWS ECR
                      prefix: acc_id.dkr.ecr.ap-southeast-1.amazonaws.com
                      api_url: https://acc_id.dkr.ecr.ap-southeast-1.amazonaws.com
                      credentials: pullsecret:argocd/mycontainerregistry-secret
                      ping: yes
                      default: true
                      insecure: no
                kind: ConfigMap
                metadata:
                  labels:
                    app.kubernetes.io/name: argocd-image-updater-config
                    app.kubernetes.io/part-of: argocd-image-updater
                  name: argocd-image-updater-config
                  namespace: argocd
                EOF

                kubectl delete pods -l app.kubernetes.io/name=argocd-image-updater -n argocd
          restartPolicy: OnFailure
```

> **_NOTE:_** ကျွန်တော်ရဲ့ cronjob မှာသုံးထားတဲ့ docker image သည် private image ဖြစ်လို့ သင်တို့ရဲ့ kubectl နဲ့  aws cli ပါတဲ့ image ကိုသုံးပေးပါ။ acc_id ဆိုတဲ့ နေရာမှာလည်း ecr repo ကိုအစားထိုးပေးပါ။


<p>တစ်ခုခု အဆင်မပြေတာရှိရင် လာမေးနိုင်ပါတယ်ဗျ။ ကျေးဇူးတင်ပါတယ်။</p>

<p style="text-align:center">
    သင်ဆရာ မြင်ဆရာ ကြားဆရာများကိုလေးစားလျှက် 🙏🙏🙏
</p>
<p style="text-align:center">
   သောင်းထိုက်ဦး (UIT)
</p>