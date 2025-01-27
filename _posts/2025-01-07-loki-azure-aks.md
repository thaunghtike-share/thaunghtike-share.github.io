---
layout:     post
title:   "Deploy the Loki Helm chart on Azure"
date:       2025-01-07 15:13:18 +0200
image: loki.png
tags:
    - devops
    - argocd
categories: DevOps
---

<p>ကျွန်တော်ဒီနေ့မှာတော့ Loki ကို Azure AKS ပေါ်မှာ azure blob storage ကို backend အဖြစ် သုံးပြီး deployလုပ်ပုံကိုရှင်းပြပေးပါမယ်။ Loki ဆိုတာ lightweight logging solution တစ်ခုဖြစ်ပြီး ပုံမှန်ဆိုရင် server ရဲ့ disk storage ကိုသုံးတာမလို့ storage ပြည့်သွားရင် ပြသနာတွေဖြစ်လာနိုင်ပါတယ်။ အဲ့ဒါမလို့ ​Azure Blob တို့ AWS S3တို့လို storage backend တစ်ခုကိုသုံးပေးသင့်ပါတယ်။ ဒါကြောင့် ကျွန်တော်က azure blob ကိုသုံးပြီး ဒိနေ့ရှင်းပြပေးသွားပါမယ်။ </p>

Loki ကို azure နဲ့ authenticate လုပ်ဖို့ နည်းလမ်း(၃)ခုရှိပါတယ်။

-   Hard coding a connection string
-   Manged identity
-   Federated token

ကျွန်တော်ဒီနေ့မှာတော့ Federated Tokenကိုသုံးပြီး Loki ကို AKS ပေါ်မှာ deploy လုပ်တာကိုတစ်ဆင့်ချင်းရှင်းပြပေးသွားမှာဖြစ်ပါတယ်။ connection string ကိုသုံးတာက production environment အတွက်အဆင်မပြေပါဘူး။

<p> အရင်ဆုံး storage accounts တစ်ခုကို create လုပ်ပေးရပါမည်။</p>

```bash
az storage account create \
--name <NAME> \
--location <REGION> \
--sku Standard_ZRS \
--encryption-services blob \
--resource-group <MY_RESOURCE_GROUP_NAME>
```
<p>ပြီးနောက် chunks နဲ့ ruler အတွက် containers တွေကို create ပေးရပါမယ်။</p>

```bash
az storage container create --account-name <STORAGE-ACCOUNT-NAME> --name <CHUNK-BUCKET-NAME> --auth-mode login && \
az storage container create --account-name <STORAGE-ACCOUNT-NAME> --name <RULER-BUCKET-NAME> --auth-mode login
```
azure portal ထဲသွားပြီး အောက်ပါအတိုင်း containers တွေကိုတွေ့ရမှာဖြစ်ပါတယ်။

![blobs](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/blobs.png)

<h2>Creating the Azure AD role and federated credentials</h2>

<p>အရင်ဆုံး ကျွန်တော်တို့ loki ကိုထည့်ချင်တဲ့ aks cluster ရဲ့ OIDC issuer url ကိုသိဖို့လိုပါတယ်။</p>

```bash
az aks show \
--resource-group <MY_RESOURCE_GROUP_NAME> \
--name <MY_AKS_CLUSTER_NAME> \
--query "oidcIssuerProfile.issuerUrl" \
-o tsv
```
<p>နောက်တစ်ဆင့်အနေနဲ့အောက်ပါအတိုင်း credentials.jsonကို create လုပ်ပေးဖို့လိုပါတယ်။ issuer နေရာမှာ အပေါ်ကရလာတဲ့ OIDC issuer url ကိုထည့်ပေးပါ။</p>

```bash
{
    "name": "LokiFederatedIdentity",
    "issuer": "<OIDC-ISSUER-URL>",
    "subject": "system:serviceaccount:loki:loki",
    "description": "Federated identity for Loki accessing Azure resources",
    "audiences": [
      "api://AzureADTokenExchange"
    ]
}
```

<p>နောက်တစ်ဆင့်အနေနဲ့ Storage Blob Contributor Role ပေးဖို့အတွက် azure ad app တစ်ခုကို အောက်ကလို create ပေးလိုက်ပါ။ ပြီးရင် app ကို federated credentials တွေ assign ချပေးရပါမယ်။</p>

```bash
 az ad app create \
 --display-name loki \
 --query appId \
 -o tsv

 az ad sp create --id <APP-ID>

 az ad app federated-credential create \
  --id <APP-ID> \
  --parameters credentials.json
 ```

<p>နောက်ဆုံးအနေနဲ့ app ကို storage roleသတ်မှတ်ပေးရပါမယ်။</p>

```bash
az role assignment create \
  --role "Storage Blob Data Contributor" \
  --assignee <APP-ID> \
  --scope /subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RESOURCE-GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE-ACCOUNT-NAME>
```

ဒါဆိုရင်တော့ ကျွန်တော်တို့တွေ loki helm chart ကို deploy လုပ်ဖို့အဆင်သင့်ဖြစ်ပါပြီ။


