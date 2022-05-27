---
layout:     post
title:   "Terraform Remote State Data Source"
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
 
 <p> ✔️ Terraform state file </p>
 <p>✔️ Terraform module</p>
 
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

```bash
-----------1_infrastructure/providers.tf-------------

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

```bash

-------------1_infrastructure/backend.tf-------------

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

<p> ✔️terraform state file ကို  s3 bucket "my-terraform-remote-state-lab-s3" ထဲမှာ "tf-infrastructure.tfstate" နာမည်နဲ့ သိမ်းထားမယ်လို့ဆိုလိုတာပါ။ key ဆိုတာကတော့ file နာမည်ပါ။ </p>
<p>✔️ encrypt ဆိုတာက state file ကို encrypt လုပ်မယ်လို့ဆိုလိုတာပါ။ </p>
<p>✔️ dynamodb ကတော့ state file locking အတွက်သုံးတာပါ။ </p>

<h2>👉 VPC module</h2>

<p> backend configuration ပြီးရင်တော့ infra module ထဲမှာ vpc resource တွေကို create လုပ်ဖို့အတွက် AWS VPC Module ကိုသုံးပါမယ်။ </p>

```bash
----------1_infrastructure/vpc.tf---------

locals {
  prefix   = "manage-alb-terraform"
  vpc_name = "${local.prefix}-vpc"
  vpc_cidr = "10.10.0.0/16"
  common_tags = {
    Environment = "dev"
    ManagedBy   = "Terraform"
  }
}

module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = local.vpc_name
  cidr = local.vpc_cidr

  azs = ["${var.aws_region}a", "${var.aws_region}b"]
  public_subnets = [
    cidrsubnet(local.vpc_cidr, 8, 0),
    cidrsubnet(local.vpc_cidr, 8, 1)
  ]

  private_subnets = [
    cidrsubnet(local.vpc_cidr, 8, 2),
    cidrsubnet(local.vpc_cidr, 8, 3)
  ]

  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true


  tags = merge(
    {
      Name = local.vpc_name
    },
    local.common_tags
  )
}
```
vpc module မှာ AZ (၂)ခု ၊ Private Subnet (၂)ခု ၊ Public Subnet (၂)ခုသုံးထားပါတယ်။ Private Subent တွေအတွက် NAT gateway တစ်ခုပါ enable လုပ်ထားပါတယ်။ 

အားလုံးပြီးသွားရင်တော့ output တွေထုတ်ခဲ့ပါမယ်။ အကြောင်းမဲ့ output တွေထုတ်တာတော့မဟုတ်ပါဘူး။ အဲ့ဒီ output တွေဖြစ်တဲ့ vpc id ၊ subnet တွေကို ALB module မှာပြန်သုံးဖို့အတွက်ပါ။

```bash
--------------1_infrstructure/outputs.tf-----------

output "prefix" {
  value       = local.prefix
  description = "Exported common resources prefix"
}

output "common_tags" {
  value       = local.common_tags
  description = "Exported common resources tags"
}

output "vpc_id" {
  value       = module.vpc.vpc_id
  description = "VPC ID"
}

output "public_subnets" {
  value       = module.vpc.public_subnets
  description = "VPC public subnets' IDs list"
}

output "private_subnets" {
  value       = module.vpc.private_subnets
  description = "VPC private subnets' IDs list"
}
```
ဒါဆိုရင်တော့ အောက်က command တွေကို သုံးပြီးတော့ infra module ကို provision လုပ်နိုင်ပါပြီ။

```bash
$ terraform init
$ terraform fmt
$ terraform validate
$ terraform plan
$ terraform apply --auto-approve
```
<h2>👉 ALB Moduel </h2>

<p>✔️ module တွေဟာ တစ်ခုနဲ့တစ်ခု ဆက်စပ်မှုမရှိတာကြောင့် module တစ်ခုကနေတစ်ခု resource တွေကို ခေါ်သုံးဖို့ဆိုရင် remote state data source ဖြစ်ဖြစ် module တွေကို initialize လုပ်ပြီးပဲဖြစ်ဖြစ်တစ်နည်းနည်းနဲ့ခေါ်သုံးရပါမယ်။ ကျွန်တော်တို့ခေါ်သုံးမယ့် child module ထဲမှာ တခြား moduel မှာပြန်သုံးနိုင်ဖို့အတွက် output တွေထုတ်ပေးရပါမယ်။ ဒါကြောင့် ကျွန်တော်အပေါ်က infra module ထဲမှာ outputs တွေထုတ်ခဲ့ပါတယ်၊ ဟုတ်ပြီ ဒါဆို child module ကဘယ်သူလဲ parent module ကဘယ်သူလဲ</p>

<p>✔️ ဒီ lab မှာဆိုရင် alb module ကို create လုပ်တဲ့အခါမှာ vpc ၊ subnet စတာတွေကို infra module ကနေပြန်သုံးမှာပါ။ ဒါကြောင့် အခေါ်ခံရတဲ့ infra module က child module ဖြစ်ပြီး alb module က parent module ဒါမှမဟုတ် calling module လို့ခေါ်ပါတယ်။ alb module အတွက်လည်း backend configuration လုပ်ပါမယ်။</p>

```bash
---------------2_alb/backend.tf---------

terraform {
  backend "s3" {
    bucket  = "my-terraform-remote-state-s3"
    key     = "terraform-alb.tfstate"
    region  = "us-west-2"
    encrypt = "true"
    dynamodb_table = "my-terraform-remote-state-dynamodb"
  }
}
```
infra module ထဲက vpc ၊ subnet စတာတွေကိုပြန်သုံးဖို့အတွက် remote state data source ကိုသုံးရပါတော့မယ်။ အဲ့လို data source ကိုသုံးမှသာလျှင် infra module ထဲက resource တွေကိုပြန်သုံးနိုင်မှာဖြစ်ပါတယ်။ infra module အတွက် state ကို သိမ်းထားတဲ့ s3 bucket ၊ key ၊ region တို့ကိုပြန်ထည့်ပေးလိုက်တာပါ။

```bash
-------------2_alb/data.tf---------

data "terraform_remote_state" "infrastructure" {
  backend = "s3"
  config = {
    bucket = "my-terraform-remote-state-s3"
    region = "us-west-2"
    key    = "tf-infrastructure.tfstate"
  }
}
````
ပြီးသွားရင်တော့ alb အတွက် လိုအပ်တဲ့ security group ကို အရင် create လုပ်ပါမယ်။

```bash
------------2_alb/alb.tf-----------

module "alb_sg" {
  source = "terraform-aws-modules/security-group/aws"

  name        = "alb-sg"
  description = "Security group for ALB"
  vpc_id      = data.terraform_remote_state.infrastructure.outputs.vpc_id

  egress_rules = ["all-all"]
}
```
ဒီနေရာမှာ vpc_id နေရာမှာ data.terraform_remote_state.infrastructure.outputs.vpc_id ဆိုပြီးတော့ infra module က output ဖြစ်တဲ့ vpc_id ကိုခေါ်သုံးလိုက်တာပါ။ ပြီးရင် alb resource ကို create လုပ်ပါမည်။

```bash
------------2_alb/alb.tf-----------

resource "aws_lb" "web" {
  name            = "alb-demo"
  subnets         = data.terraform_remote_state.infrastructure.outputs.public_subnets
  security_groups = [module.alb_sg.security_group_id]

  tags = {
      Name = "alb-demo"
  }
}
```
ဒီနေရာမှာလည်း subnet နေရာကို infra moudle က public subnet တွေကို remote state data နဲ့ခေါ်သုံးလိုက်ပါတယ််။ ဒီလောက်ဆို Terraform ရဲ့ remote state data source အကြောင်းကိုနားလည်သဘောပေါက်လို့မယ်ထင်ပါတယ်။ အားလုံကိုကျေးဇူးတင်ပါတယ်။

<h2> References </h2>

<ul>
    <li><a href="https://www.terraform.io/language/state/remote-state-data">https://www.terraform.io/language/state/remote-state-data</a></li>
</ul>
<p>
သင်ဆရာ မြင်ဆရာ ကြားဆရာများကိုလေးစားလျှက် 🙏🙏🙏
</p>
<p>
သောင်းထိုက်ဥိး (UIT)
</p>
