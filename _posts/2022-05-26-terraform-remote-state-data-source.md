---
layout:     post
title:   "The terraform_remote_state Data Source"
date:       2022-05-26 15:13:18 +0200
image: 36.png
tags:
    - Terraform
    - IAC
categories: Terraform
---

<h2>👉 Introduction</h2>

 <p>✔️ ကျွန်တော်ဒီနေ့မှာတော့ Terraform ရဲ့ remote state data source အကြောင်းကိုနားလည်သလောက်လေးရှင်းပြပေးမှာဖြစ်ပါတယ်။ remote state data source ဆိုတာက ကျွန်တော်တို့ backend တစ်ခုခုမှာ သိမ်းထားတဲ့ state file ထဲ data တွေကို အခြား Terraform module တွေမှာပြန်သုံးဖို့အတွက်သုံးတာဖြစ်ပါတယ်။ </p>
 
 <h2>👉 Prerequities </h2>
 
 ✔️ Terraform state file
 ✔️ Terraform module
 
 <h2>👉 State File ဆိုတာဘာလဲ?</h2>
 
 <p> ✔️ Terraform state file ဆိုတာက ကျွန်တော်တို့ provision လုပ်လိုက်တဲ့ infrastructure တစ်ခုလုံးနဲ့သက်ဆိုင်တဲ့ information တွေပါဝင်ပါတယ်။ ဒါကြောင့် အဲ့ဒီ state file တွေကို safe ဖြစ်တဲ့ နေရာတစ်ခုခု ဆိုလိုတာက backend တစ်ခုခုမှာ သိမ်းထားဖို့လိုပါတယ်။ ဘာလို့လဲဆိုရင် sensitive data တွေ ဖြစ်တဲ့ DB password တွေ user password တွေပါဝင်နေလို့ပါ။ local backend ကိုသုံးတဲ့အခါမျိုးမှာဆိုရင် state file ကိူ plan-text json format နဲ့ store လုပ်ပါတယ်။ </p>
 
 <p>✔️ ဒါကြောင့် security အတွက်ဆို remote backend တွေကိုသုံးဖို့အကြံပြုကြပါတယ်။ Remote Backend တွေကလည်းအမျိုးမျိုးရှိပါတယ်။ s3၊ Terraform Cloud စသည်ဖြင့်ပေါ့။ ကျွန်တော်ကတော့ Terraform Cloud နဲ့ s3 backend ကိုပဲအသုံးများပါတယ်။ Terraform Cloud ဆိုရင် state ကိုအမြဲ encrypt လုပ်ပေးပါတယ်။ ဒီလောက်ဆို backend အကြောင်းကို အနည်းငယ်သဘောပေါက်လောက်မယ်ထင်ပါတယ်။</p>
 
<h2>👉 Examples</h2> 

ကျွန်တော် remote state data source အကြောင်းကိုပိုမြင်သွားအောင် lab လေးတစ်ခုနဲ့ ရှင်းပြပါမယ်။

```bash
├── 1_infrastructure
│   ├── backend.tf
│   ├── main.tf
│   ├── outputs.tf
│   ├── providers.tf
│   ├── variables.tf
│   └── vpc.tf
├── 2_alb
│   ├── README.md
│   ├── alb.tf
│   ├── backend.tf
│   ├── main.tf
│   ├── outputs.tf
│   ├── providers.tf
│   ├── usage.md
│   └── variables.tf
├── README.md
```
အပေါ်က tree မှာဆိုရင် module  ၂ခုကို တွေ့ရမှာဖြစ်ပါတယ်။ ပထမတစ်ခုကတော့ infrastructure module ဖြစ်ပြီး ဒုတိယတစ်ခုကတော့ alb module ဖြစ်ပါတယ်။ infra module မှာတော့ ဥပမာ VPC နဲ့ဆိုင်တဲ့ resource တွေပါဝင်ပြီးတော့ ALB module မှာတော့ application load balancer နဲ့ဆိုင်တဲ့ resource တွေပါဝင်ပါတယ်။

<h2>👉 Infrastructure Module</h2>

infra module ထဲမှာ ပထမဦးဆုံး provider နဲ့ backend configuration ကိုအရင်ချပါမယ်။ provider ကတော့ aws provider ကိုပဲ example အနေနဲ့ သုံးထားပါတယ်။ အောက်ကတော့ AWS provider အတွက် Terraform code ပါ။

```yaml
************1_infrastructure/providers.tf****************

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.9"
    }
  }
}

# Defining AWS provider
provider "aws" {
  region = var.aws_region
}
```
ပြီးသွားရင်တော့ backend configuration ရေးပါမယ်။ s3 backend ကို remote backend အနေနဲ့ သုံးလိုက်ပါမယ်။

```yaml

***********1_infrastructure/backend.tf****************

terraform {
  backend "s3" {
    bucket  = "my-terraform-remote-state-lab-s3"
    key     = "tf-infrastructure.tfstate"
    region  = "us-west-2"
    encrypt = "true"
    dynamodb_table = "my-terraform-remote-state-dynamodb"
  }
}
```
backend configuration ထဲမှာသုံးထားတာကတော့ရိုးရှင်းပါတယ်။ 

✔️terraform state file ကို  s3 bucket "my-terraform-remote-state-lab-s3" ထဲမှာ "tf-infrastructure.tfstate" နာမည်နဲ့ သိမ်းထားမယ်လို့ဆိုလိုတာပါ။ key ဆိုတာကတော့ file နာမည်ပါ။ 
✔️ encrypt ဆိုတာက state file ကို encrypt လုပ်မယ်လို့ဆိုလိုတာပါ။
✔️ dynamodb ကတော့ state file locking အတွက်သုံးတာပါ။

backend configuration ပြီးရင်တော့ infra module ထဲမှာ vpc resource တွေကို create လုပ်ဖို့အတွက် <a href="https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest" VPC module</a> ကိုသုံးပါမယ်။
