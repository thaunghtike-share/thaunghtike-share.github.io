---
layout:     post
title:   "Gitlab Runner Autoscaling With GCP Preemptible VMs"
date:       2022-05-14 15:13:18 +0200
image: gitlab_runner_gcp.png
tags:
    - GCP
    - CICD
categories: CICD
---

<h2> Gitlab Runner Autoscaling With GCP Preemptible VMs </h2>

<h3> 👉 Introduction </h3>

ကျွန်တော်ဒီနေ့ရေးမှာကတော့ google cloud က preemptible vms တွေနဲ့ gitlab runner ကို autoscale လုပ်တာပဲဖြစ်ပါတယ်။ ဒီ topic က ကျွန်တော့်သူငယ်ချင်းတစ်ယောက် ရေးထားတာလေးကိုသဘောကျလို့ ပြန်ရေးလိုက်တာပါ။ သူကတော့ aws spot instances တွေနဲ့ ရှင်းပြထားပါတယ်။ အမှန်အတိုင်း ဝန်ခံရရင် gitlab runner autoscaling အကြောင်းကို တစ်ခါမှမစဥ်းစားမိခဲ့ပါဘူး။ ဘာလို့လဲဆိုရင် ကျွန်တော့်ရဲ့အလုပ်မှာက autoscale သုံးဖို့လောက်အထိ pipelines တွေမများလို့ပါ။ ဟုတ်ပြီ.. ဒီနေ့ကျွန်တော်စာဖတ်လို့နားလည်သလောက်ကို demo တစ်ခုနဲ့ ရှင်းပြလိုက်ပါတယ်။

<h2> 👉 Why Runner Needs Autoscaling </h2>

Runners တွေဟာ ဘာလို့ autoscale လုပ်ဖို့လိုတာလဲ။ များသောအားဖြင့် ကျွန်တော်တို့တွေဟာ runners တွေကို docker executor နဲ့ပဲ register လုပ်ကြပါတယ်။ pipeline ထဲမှာ run မယ့် jobs ကနည်းရင် သိပ်မသိသာပါဘူး။ CI CD pipeline တွေကို လူတွေအများကြီးက ပြိုင်တူ run တဲ့အခါ docker နဲ့ register လုပ်ထားတဲ့စက်က CPU overload ဖြစ်ပြီး ကောင်းကောင်းအလုပ်မလုပ်နိုင်တော့ပါဘူး။ ဒါဆို cloud ပေါ်က compute engine ကို scale up လုပ်ဖို့ဆိုရင်လည်း စဥ်းစားစရာဖြစ်လာပြန်ရော။ runner အဖြစ်ပဲသုံးမယ့် vm အတွက် ဘာလို့ တစ်လ ကို ပိုက်ဆံတွေအများကြီးပေးနေရမှာလဲ။ runner မသုံးတဲ့အချိန်တွေမှာ အလကားပိုက်ဆံပေးနေရမှာပါ။ အဲ့ဒီအတွက် ဖြေရှင်းဖို့ အဖြေကတော့ spot instances တွေနဲ့ autoscale လုပ်ခြင်းပဲဖြစ်ပါတယ်။ spot instance ဆိုတာ normal instance တွေထက်  70-90% လောက်အထိ စျေးသက်သာပါတယ်။ ကိုယ်လိုတဲ့အချိန်မှာသာ run နိုင်ပြီး ပြီးသွားရင်ပြန်ပိတ်ခဲ့လို့ရပါတယ်။ အထူးသဖြင့် batch job တွေ run တဲ့အခါသုံးကြပါတယ်။

<h2>👉 Runner Manager Setup</h2>

Runner manager လို့ပဲခေါ်ခေါ် bastion host လို့ပဲခေါ်ခေါ် အတူတူပါပဲ။ သူရဲ့အဓိက တာဝန်က runner တွေကို scale in ၊ scale out လုပ်ပေးဖို့ပါပဲ။ ဒါဆိုရင် ပထမဆုံးအနေနဲ့ runner manager အတွက် compute engine တစ်ခုကို create လုပ်ပေးရပါမယ်။ ec2-micro လောက်ဆို လုံလောက်ပါပြီ။ runner manager ကနေ runner vm တွေကို create လုပ်နိုင်ဖို့ Compute API ကို access ရအောင် Access Scopes မှာရွေးပေးပါ။ ssh နဲ့ တစ်ခြား services တွေအတွက် firewall rules တွေသတ်မှတ်ပေးပါ။ အားလုံးပြီးသွားရင်တော့ အောက်က လို vm တစ်ခု ရလာပါလိမ့်မယ်။

![vm](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/vm.png)

<h2>👉 Create Bucket</h2>

vm create ပြီးရင် vm ထဲကို ssh ဝင်ပြီး runner register ရပါတော့မယ်။ register မလုပ်ခင်မှာ autoscaling အတွက်လိုအပ်တာတွေလုပ်ပေးရပါဦးမယ်။ runner config ထဲမှာ runner cache တွေသိမ်းဖို့အတွက် လိုအပ်တဲ့ bucket တစ်ခု create ရပါမယ်။ ဒါကြောင့် bucket တစ်ခုကို create လိုက်ပါမယ်။ bucket နာမည်ကိုတော့ tho-gcs လို့ပေးလိုက်ပါတော့မယ်။

![gcs](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/gcs.png)

bucket create ပြီးရင် runner config ထဲမှာ compute engine တွေ scale လုပ်ဖို့ အတွက်ရယ် cache တွေ bucket ထဲမှာ သိမ်းဖို့ အတွက်ရယ် api တွေနဲ့ access ရဖို့ service account တစ်ခုဆောက်ရပါမယ်။ အဲ့ service account ရဲ့ key ကို download ပြီး runner config ထဲမှာ ထည့်ပေးရပါမယ်။ ဒါမှသာ runner အတွက် vms တွေ scale in ၊ scale out လုပ်နိုင်မှာပါ။

![sa](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/sa.png)

<h2>👉 Create Service Account</h2>

service account create ပြီးရင် ဘေးကအစက် (၃)စက်ကနေ manage key ကိုနှိပ်ပြီး key ကို download လုပ်ထားပေးပါ။ အားလုံးပြီးသွားပြီဆိုရင် runner manager vm ထဲကို ssh ဝင်ပြီး runner register လုပ်ပါတော့မယ်။
gitlab runner ကို install လုပ်ပေးပါ။

```bash
sudo curl -L --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64"
sudo chmod +x /usr/local/bin/gitlab-runner
```
ပြီးရင် docker နဲ့ docker machine ကိုအရင် install လုပ်ပေးရပါမယ်။ docker က runner ကို register ဖို့လိုအပ်တာဖြစ်ပြီး docker-machine က gcp vm တွေကို create remove လုပ်ပေးမှာပါ။

```bash
sudo su - && apt update -y && apt install docker.io -y && systemctl restart docker

base=https://github.com/docker/machine/releases/download/v0.16.0 \
  && curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine \
  && sudo mv /tmp/docker-machine /usr/local/bin/docker-machine \
  && chmod +x /usr/local/bin/docker-machine
```
<h2>👉 Register Runner</h2>

အားလုံးပြီးသွားတဲ့အခါ gitlab ထဲကိုသွားပါ မိမိ runner ကို register လုပ်ချင်တဲ့ project (သို့) group အောက်က settings >> CI CD >> runners အထဲမှာ expand ကို click လိုက်ရင် အောက်ကလို runner token ရလာပါလိမ့်မယ်။ token ရရင် runner manager vm ထဲမှာ runner-manager register command နဲ့ runner ကို register လုပ်ပါ။ executor မှာ docker+machine  ကိုရွေးပါ။

```bash
root@runner-manager:~# gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=15064 revision=c1edb478 version=14.0.1
Running in system-mode.

Enter the GitLab instance URL (for example, https://gitlab.com/):
https://gitlab.com/
Enter the registration token:
WM4hpNGeEPxyoH1s79dC
Enter a description for the runner:
[runner-manager]:
Enter tags for the runner (comma-separated):
Registering runner... succeeded                     runner=WM4hpNGe
Enter an executor: docker+machine, docker-ssh+machine, custom, docker-ssh, parallels, virtualbox, docker, shell, ssh, kubernetes:
docker+machine
Enter the default Docker image (for example, ruby:2.6):
alpine:latest
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```
ပြီးသွားရင်  gitlab-runner verify နဲ့ ကြည့်လိုက်ပါ။ runner တစ်ခု active ဖြစ်နေတာကိုတွေ့ရပါလိမ့်မယ်။ ဒါက ရိုးရိုး runner register အပိုင်းပဲရှိပါသေးတယ်။ autoscale အတွက်ဆို /etc/gitlab-runner/config.toml file ကိုအောက်ကအတိုင်းပြင်ပေးရပါမယ်။ client_secret.json ဆိုတာ create ခဲ့တဲ့ service account ရဲ့ keys ပါ။ bucket ကတော့ create ခဲ့တဲ့ bucket ထည့်ပေးပါ။ အရင်ဆုံး /etc/gitlab-runner အောက်မှာ client_secret.jsonဆိုတဲ့ file ဆောက်ပေးပါ download ခဲ့တဲ့ keys ကိုထည့်လိုက်ပါ။ +x permissions ပေးလိုက်ပါ။

```yaml
vim /etc/gitlab-runner/client_secret.json
chmod +x /etc/gitlab-runner/client_secret.json
```
/etc/gitlab-runner/config.toml ကိုလည်းအောက်ပါအတိုင်းedit လိုက်ပါ။ concurrent ဆိုတာ တစ်ပြိုင်ထဲ job ၈ခုထိ run နိုင်တယ်လို့ပြောတာပါ။ docker machine ထဲမှာက scaling လုပ်တဲ့အခါ ubuntu 20.04 ကိုသုံးမယ်ပြောတာဖြစ်ပြီး preemptible ကို true ပေးထားတာက spot instance အဖြစ်သုံးမယ်လို့ပြောထားတာဖြစ်ပါတယ်။

```bash
concurrent = 8
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "bastion"
  url = "https://gitlab.com/"
  token = "WM4hpNGeEPxyoH1s79dC"
  executor = "docker+machine"
  [runners.custom_build_dir]
  [runners.cache]
    Type = "gcs"
    Path = "gitlab-runner"
    Shared = false

    [runners.cache.s3]
    [runners.cache.gcs]
      BucketName = "tho-gcs"
      CredentialsFile = "/etc/gitlab-runner/client_secret.json"
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
  [runners.machine]
    IdleCount = 0
    IdleTime = 30
    MachineDriver = "google"
    MachineName = "auto-scale-runner-%s"
    MachineOptions = [
      "google-project=clever-circlet-317904",
      # Depending on your requirements, choose another instance
      "google-machine-type=e2-highcpu-2",
      # When running the forked docker-machine, you should use cos-stable
      "google-machine-image=ubuntu-os-cloud/global/images/ubuntu-2004-focal-v20210510",
      "google-preemptible=true",
      "google-zone=europe-north1-a",
      "engine-registry-mirror=https://mirror.gcr.io"
    ]
```
ဒါဆိုရင် ကျွန်တော် gitlab ထဲက project ထဲသွားပြီး .gitlab-ci.yml file ကို အောက်ကအတိုင်း create လိုက်ပါမယ်။

![gitlab](https://raw.githubusercontent.com/thaunggyee/thaunggyee.github.io/master/img/ci.png)

မကြာခင် pipeline ထဲမှာ job တစ်ခု run နေပါလိမ့်မယ်။ pending ဖြစ်နေရင် gitlab-runner --debug run ကို run  ပေးပါ။ running state ရောက်တဲ့အခါ runner logs တွေကို ဝင်ကြည့်လို့ရပါတယ်။ job ပြီးသွားတဲ့အခါ အောက်ကလိုတွေ့ရပါလိမ့်မယ်။

![job](https://raw.githubusercontent.com/thaunggyee/thaunggyee.github.io/master/img/job.png)

complete ဖြစ်သွားတဲ့ job ကို အောက်ကပုံထဲမှာတွေ့ရသလို runner-scale vm ကနေ execute လုပ်ပေးခဲ့တာဖြစ်ပါတယ်။ runner vm တစ်ခု ကို runner manager vm ကနေ scale out လုပ်ပေးလိုက်တာပါ။

![scalevm](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/scale.png)

<h2>👉 Summary</h2>

job complete ဖြစ်ပြီး second 30 ကြာတော့ scale out ဖြစ်ခဲ့တဲ့ vm ဟာ auto teminated ဖြစ်သွားပါတယ်။ ဒီလိုနဲ့ job တွေ run တိုင်း vm တစ်ခု ထွက်လာမှာဖြစ်ပြီး job completed သွားတဲ့အခါ auto scale in ပြန်ဖြစ်သွားမှာဖြစ်ပါတယ်။ ဒီနည်းနဲ့ runner ကို docker executor နဲ့ deploy ခဲ့တဲ့ host ကိုလည်း cpu overload ဖြစ်ခြင်းကကာကွယ်နိုင်သလို normal instance တွေ သုံးပြီး runner တွေ create တာထက်အသုံးစရိတ်ကိုလည်း သက်သာစေပါတယ်။
ပိုက်ဆံတွေလည်းမလိုအပ်ပဲ အများကြီးကုန်စရာမလိုတော့ပါဘူး။ အဆုံးထိဖတ်ပေးခဲ့ကြတဲ့ သူအားလုံးကိုကျေးဇူးတင်ပါတယ်။

<h2>👉 References </h2>
<ul>
    <li><a href="https://medium.com/@webnugget/setup-a-costeffective-autoscaling-ci-solution-with-gitlab-ci-5a6ac52a5baa">https://medium.com/@webnugget/setup-a-costeffective-autoscaling-ci-solution-with-gitlab-ci-5a6ac52a5baa</a></li>
    <li><a href="https://blog.dopevs.cloud/2019-08-02-Gitlab-Runner-Autoscaling-with-aws-spot-instances/">https://blog.dopevs.cloud/2019-08-02-Gitlab-Runner-Autoscaling-with-aws-spot-instances</a></li>
</ul>

<p style="text-align:center">
    သင်ဆရာ မြင်ဆရာ ကြားဆရာများကိုလေးစားလျှက် 🙏🙏🙏
</p>
<p style="text-align:center">
   သောင်းထိုက်ဦး (UIT)
</p>
