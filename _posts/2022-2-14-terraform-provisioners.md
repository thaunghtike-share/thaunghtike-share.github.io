---
layout:     post
title:   "Terraform Provisioners"
date:       2022-02-14 15:13:18 +0200
image: 36.png
tags:
    - Terraform
    - IAC
categories: Terraform
---

ကျွန်တော်တို့ Infrastructure ကို provision လုပ်ဖို့အတွက် Terraform ကို သုံးကြတဲ့အခါမှာ provisioners တွေက အမြဲလိုလိုသုံးရလေ့ရှိပါတယ်။ provisioner ဆိုတာက ကျွန်တော်တို့ရဲ့ local ဒါမှမဟုတ် remote စက်တွေပေါ်မှာ action တွေ scripts တွေကို execute လုပ်ဖို့သုံးချင်တာပါ။ဒီနေရာမှာ local ဆိုတာ Terraform ကို run နေတဲ့ ကျွန်တော်တို့ရဲ့ machine ဖြစ်ပြီး remote ဆိုတာကတော့ Terraform နဲ့ provision လုပ်လို့ရလာတဲ့ resource တွေကိုဆိုလိုတာပါ။

Provisioners are used to execute scripts on a local or remote machine as part of resource creation or destruction. Provisioners can be used to bootstrap a resource, cleanup before destroying, run configuration management, etc.

<h2>👉 Provisioners အမျိုးအစား</h2>

ယေဘုယျအားဖြင့်တော့ Provisioner အမျိုးအစား (၂) ခုရှိပါတယ်။
<ul>
    <li>Generic Provisioners ( file, local-exec, and remote-exec )</li>
    <li>Vendor Provisioners ( chef, puppet, ansible, etc )</li>
</ul>

ကျွန်တော်ကတော့ generic provisioners တွေအကြောင်းကိုပဲ ရေးသွားမှာပါ။ ဒါဆိုအခု တစ်ခုချင်း အကြောင်းကိုရှင်းပြပါ့မယ်။ privisioner တွေအကြောင်းကိုမရှင်းခင်မှာ connection block အကြောင်းကိုအရင်ရှင်းပြပါမယ်။

<h2>👉 Connection Block အကြောင်း</h2>

file နဲ့ remote-exec provisioner တွေမှာ remote resources တွေကို access လုပ်ဖို့အတွက် connection block ထဲမှာ သတ်မှတ်ပေးကြရပါတယ်။ connection type မှာ SSH နဲ့ WinRM (၂)ခုကို support လုပ်ပါတယ်။ connection block မှာတော့ remote machine ရဲ့ host, username, private_key တွေကို သတ်မှတ်ပေးရပါမယ်။

```bash
provisioner "file" {
  source      = "conf/myapp.conf"
  destination = "/etc/myapp.conf"

  connection {
    type     = "ssh"
    user     = "root"
    password = "${var.root_password}"
    host     = "${var.host}"
  }
}
```

<h2>👉 The self Object</h2>

Provisioner ရဲ့ connection block ကိုသတ်မှတ်တဲ့အခါမှာ parent block ရဲ့ IP address စတာတွေကို name နဲ့ ပြန်ခေါ်မသုံးပဲ self-object နဲ့သုံးပါတယ်။ ဥပမာ - remote EC2 ရဲ့ public IP ကို သတ်မှတ်ဖို့ဆို self.public_ip ဆိုပြီးသတ်မှတ်နိုင်ပါတယ်။

<h2>👉 File Provisioner  တွေအကြာင်း</h2>

File provisioners တွေဆိုတာ အလွယ်ပြောရရင်တော့ local machine ကနေ remote ကို files တွေကို copy လုပ်ဖို့သုံးတာဖြစ်ပါတယ်။ File Provisioner တွေမှာ source နဲ့ destination ဆိုပြီးသတ်မှတ်ပေးရပါတယ်။ source ကတော့ local machine က copy ကူးမယ့် file ရဲ့ path ဖြစ်ပြီး destination ကတော့ remote machine က path ဖြစ်ပါတယ်။

The file provisioner is used to copy files or directories from the machine executing Terraform to the newly created resource. The file provisioner supports both ssh and WinRM type connections.

```bash
provisioner "file" {
  source      = "conf/myapp.conf"
  destination = "/etc/myapp.conf"

  connection {
    type     = "ssh"
    user     = "root"
    password = "${var.root_password}"
    host     = "${var.host}"
  }
}
```

<h2>👉 Local-Exec Provisioner</h2>

Local-Exec provisioner တွေကတော့ Terraform နဲ့ resources တွေ create ပြီးတဲ့အခါမှာ ကိုယ့်ရဲ့ local machine ကနေ command တွေကို execute လုပ်ဖို့အတွက်သုံးတာဖြစ်ပါတယ်။ ဥပမာ - kubernetes cluster တွေကို kubectl နဲ့ သုံးတာမျိုးဖြစ်ပါတယ်။ သူ့မှာတော့ connection block သတ်မှတ်ပေးဖို့မလိုပါဘူး။

```bash
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "echo ${self.private_ip} >> private_ips.txt"
  }
}
```

<h2>👉 Remote-Exec Provisioner</h2>

Terraform နဲ့ resource တွေကို create လုပ်ပြီးတဲ့အခါမှာ အဲ့ဒီ resource မှာ တစ်ခုခုကို execute လုပ်ဖို့သုံးပါတယ်။ ဥပမာ - configuration management တွေ run ဖို့ bash scripts တွေ run ဖို့အတွက်သုံးနိုင်ပါတယ်။

```bash
resource "aws_instance" "web" {
  # ...

  # Establishes connection to be used by all
  # generic remote provisioners (i.e. file/remote-exec)
  connection {
    type     = "ssh"
    user     = "root"
    password = var.root_password
    host     = self.public_ip
  }

  provisioner "remote-exec" {
    inline = [
      "puppet apply",
      "consul join ${aws_instance.web.private_ip}",
    ]
  }
}
```

<h2>👉 Destroy-Time Provisioners</h2>

ပုံမှန်အားဖြင့်ဆို provisioner တွေက resource တွေကို creation လုပ်ပြီးတဲ့အခါမှာ run လုပ်တာပါ။ ဆိုတော့ destroy-time ဆိုတော့ရှင်းပါတယ်။ resource တွေကို terraform destroy နဲ့ destroy လုပ်ချင်တဲ့အခါမျိုးမှာ သုံးဖို့ပါ။ provisioner block မှာ when = destroy ဆိုပြီးသတ်မှတ်ပေးရင်ရပါတယ်။

```bash
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Destroy-time provisioner'"
  }
}

```

<h2>👉 Failure Behavior</h2>

ပုံမှန်အားဖြင့် provisioner error တက်ပြီး fail သွားရင် terraform apply ကလည်း fail ဖြစ်သွားမှာပါ။ on_failure = continue ဆိုတာက provisioner fail သွားလည်း terraform apply ကဆက်လက်အလုပ်လုပ်သွားမှာပါ။

```bash
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command    = "echo The server's IP address is ${self.private_ip}"
    on_failure = continue
  }
}
```
ကဲ.. စာလည်း အတော်ရှည်သွားပြီ။ ဒီလောက်ဆို Terraform Provisioners တွေအကြောင်းကိုနားလည်မယ့်လို့ထင်ပါတယ်။ အဆုံးထိဖတ်ရှူပေးကြသူများအားလုံးကျေးဇူးတင်ပါတယ်။ နောက်လည်း ဒီလို sharing လေးတွေရေးသွားပါဦးမယ်။


<h2>👉 Reference</h2>

<ul>
    <li><a href="https://www.terraform.io/language/resources/provisioners/syntax?fbclid=IwAR3dfbfsRCbT1Cmqp8MsO5TTENANMGVnuY3rhKExpjKT6gneSoeKJ4fqMlA">https://www.terraform.io/language/resources/provisioners/syntax?fbclid=IwAR3dfbfsRCbT1Cmqp8MsO5TTENANMGVnuY3rhKExpjKT6gneSoeKJ4fqMlA</a> </li>
</ul>

<p style="text-align:center">
    သင်ဆရာ မြင်ဆရာ ကြားဆရာများကိုလေးစားလျှက် 🙏🙏🙏
</p>
<p style="text-align:center">
   သောင်းထိုက်ဦး (UIT)
</p>
