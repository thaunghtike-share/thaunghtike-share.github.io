---
layout:     post
title:      "Lxc & Ansible"
subtitle:   "Introduction To Ansible Ad-Hoc Commands & Playbooks ( PART-II)"
date:       2021-07-09 3:00:00
author:     "Thaung Htike Oo"
header-img: img/black.jpg
catalog: true
tags:
    - Ansible
    - IAC
---

<h2> PART-I </h2>
 
PART-I မှာတုန်းက lxc container (၃)ခုကိုသုံးပြီး ansible master တစ်ခုရယ် worker နှစ်ခုရယ်ကို configure လုပ်ခဲ့ပြီးပါပြီ။ PART-II မှာတော့ ad-hoc commands တွေနဲ့ playbook အတွေအကြောင်းကိုအတူတူ လေ့လာလိုက်ရအောင်နော်။

<h2> Ad-Hoc Commands </h2>

Plyabook တွေမရေးခင်မှာကျွန်တော်တို့တွေ ad-hoc command ဆိုတာဘာလဲအရင်လေ့လာလိုက်ကြရအောင်ဗျာ။ ad-hoc ဆိုတာ worker nodes တွေမှာတစ်ခုခု run ဖို့ရေးလိုက်တဲ့ short command လေးပဲဖြစ်ပါတယ်။ တစ်နည်းအားဖြင့် worker တစ်ခုပေါ်မှာ run မယ့် linux command ကို မိမိအလိုရှိတဲ့ worker တွေပေါ်မှာ run လိုက်တာပဲဖြစ်ပါတယ်။ ဥပမာ- node01 နဲ့ node02 ၂ခုလုံးမှာ df -hT ဆိုတဲ့ command ကို run မယ်ဆို node တွေထဲကို login ပြီး terminal ကနေ df -hT ဆိုပြီးသွားရိုက်ထည့်ရမှာဖြစ်တယ်။ ad-hoc command ဆိုတာ အဲ့လို node တစ်ခုချင်းစီကို terminal ကနေ မဝင်ပဲ master node ကနေပဲ command ကို run ပြီး server တွေအပေါ်မှာ သက်ရောက်မှုရှိစေတာပဲဖြစ်ပါတယ်။ 

ansible မှာ module တွေလည်းရှိပါသေးတယ်။ packages တွေ install ဖို့ apt ၊ yum ၊ dnf စတဲ့ module တွေ ၊ file တွေ directory တွေအတွက် file module ၊ copy ဖို့အတွက် copy module ၊ user တွေ create delete ဖို့ user module ဆိုပြီး သတ်မှတ်ထားပါတယ်။ အခြား module တွေလည်းအများကြီးကျန်ရှိနေပါသေးတယ်။ ad-hoc commands အနည်းငယ်ကို လေ့လာကြည့်ရအောင်။ 

shell module ဟာ ad-hoc command တွေမှာ defautl module ဖြစ်တယ်။ module ကို -m နဲ့ ကိုယ်စားပြုတယ်။ အောက်မှာတွေ့ရတဲ့ command ဟာ worker nodes တွေရဲ့ df -hT ကိုလှမ်းကြည့်တာဖြစ်တယ် ansible အနောက်က all ဆိုတာကတော့ all worker nodes ကိုဆိုလိုခြင်းဖြစ်တယ်။ inventory ထဲမှာသတ်မှတ်ခဲ့တဲ့ webserver တွေမှာပဲ run ချင်ရင် all နေရာမှာ webserver လို့ထည့်ပြီး dbserver တွေကိုပဲ run ချင်ရင် dbserver လို့ထည့်ပါတယ်။ shell နဲ့ command module နှစ်ခုလုံးဟာအတူတူပါပဲ။ linux command တွေကို run ဖို့သုံးတာပါ။
```bash
ansible -m shell -a 'df -hT'
```
နောက်တစ်ခုက haproxy ကို worker အကုန်လုံးမှာ install မှာဖြစ်ပြီး -b --become-user=root က root အနေနဲ့ run မယ်လို့ဆိုလိုတာပါ။
```bash
ansible all -m shell -b --become-user=root -a 'apt install haproxy -y'
```
နောက်တစ်ခုကတော့ file module ဖြစ်ပြီး file တွေ directory တွေကို create လုပ်ခြင်း delete လုပ်ခြင်း permission ပြောင်းခြင်းစသည်တို့အတွက်သုံးပါတယ်။ stateက file လား directory လားဆိုတာသတ်မှတ်တာပါ file ဆို state=touch ဖြစ်ပြီး directory ဆို directory ဖြစ်ပါတယ်။ အောက်က cmd ကတော့ webserver nodes တွေမှာ test ဆိုတဲ့ file ကို 777 permission နဲ့ /root အောက်မှာ create လုပ်မှာဖြစ်ပါတယ်။
```bash
ansible webserver -m file -a 'dest=/root/test mode=777 state=touch' 
```
ကဲ.. ဒီလောက်ဆို ad-hoc commands တွေအကြောင်းကိုနားလည်မယ်ထင်ပါတယ်။ တစ်ခြား module တွေကိုတော့ ကိုယ်တိုင် လေ့လာကြည့်ကြဖို့တိုက်တွန်းပါတယ်။

<h2> Ansible Playbook </h2>

Playbook ဆိုတာ server configuration တွေကို လုပ်မယ့် organized unit of scripts တစ်ခုပဲဖြစ်ပါတယ်။ ad-hoc command တွေဟာ short command တစ်ခုသာဖြစ်ပြီး ကျွန်တော်တို့ server တွေမှာ configuration လုပ်ဖို့ဆို playbook တွေ ansible-galaxy တွေ roles တွေကိုသုံးကြရပါတယ်။ ဒီ lab မှာတော့ ansible roles အကြောင်းမပါသေးပါဘူး။ playbook မှာလည်းပဲ module တွေကို အဓိကထားပြီးသုံးကြပါတယ်။ အောက်က sample playbook လေးကို လေ့လာကြည့်ရအောင်။
```yaml
---
- hosts: all
  become: yes
  pre_tasks:
    - name: "update cache"      
      apt:
        update_cache: yes
  tasks:
    - name: "Install Boxes"
      apt:
        name:
          - boxes
    - name: "Install figlet"
      apt:
         name: 
           - figlet
```
playbook က boxes နဲ့ figlet ဆိုတဲ့ tool နှစ်ခုကို worker တွေမှာ install လုပ်ဖို့ရေးထားတာပါ။ package management ဆိုတော့ apt ၊ yum ၊ dnf တစ်ခုခုကိုသုံးပြီး lab မှာ worker တွေက ubuntu တွေဖြစ်လို့ apt ကိုသုံးပါမယ်။ host all ဆိုတာကတော့ worker အားလုံးပေါ်လို့ဆိုလိုတာ။ pre tasks ဆိုတာ tasks မ run ခင်မှာ run ချင်တာတွေဆို initialize လုပ်ပြီးသုံးတာ update_cache သည် apt update -y နဲ့အတူတူပါပဲ။
playbook ကို run လိုက်ရင်တော့  အောက်ကလိုတွေ့ရပါမယ်။
```bash
root@master:~# ansible-playbook apt.yaml 

PLAY [TESTING ANSIBLE APT MODULE] ********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.28.212.239]
ok: [10.28.212.91]

TASK [install boxes] *********************************************************************************************************************************
changed: [10.28.212.239]
changed: [10.28.212.91]

TASK [install figlet] ********************************************************************************************************************************
changed: [10.28.212.91]
changed: [10.28.212.239]

PLAY RECAP *******************************************************************************************************************************************
10.28.212.239              : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.28.212.91               : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
install ပြီးတဲ့အခါမှာ ok=3 ဆိုပြီးတွေ့ရမှာပါ။ worker node တစ်ခုမှာသွားပြီး install လုပ်ပြီးတာ သေချာအောင်စမ်းကြည့်ရအောင်။
```bash
root@node01:~# figlet hello
 _          _ _       
| |__   ___| | | ___  
| '_ \ / _ \ | |/ _ \ 
| | | |  __/ | | (_) |
|_| |_|\___|_|_|\___/ 
```
နောက်ထပ် playbook တစ်ခုကတော့ apache webserver ကို webserver တွေပေါ်မှာပဲ install လုပ်မှာဖြစ်ပါတယ်။
```yaml
---
- name: Intall apache2
  hosts: webserver
  pre_tasks:
       - name: upate packages
         apt:
           update_cache: yes
  become: true
  tasks:
       - name: install apache2
         apt:
           name: apache2
           state: latest
         notify:
             - restart apache2
         become: true
  handlers:
       - name: restart apache2
         service:
              name: apache2
              state: restarted
```
apt မှာ state တွေရှိပါတယ် present ဆိုတာ install ပြီးရင် skip မယ် မရှိရင် install မယ် absent state က uninstall တာကို ဆိုလိုတာပါ။ ဒီ playbook မှာတော့ apache2 ကို install မှာဖြစ်တယ်။ notify နဲ့ handler ဆိုတာက apache ကို install လုပ်ပြီးတဲ့အခါမှာ notify လုပ်မယ် notify မှာသတ်မှတ်ခဲ့တဲ့ restart apache2 ကို handlers မှာ သွားအလုပ်လုပ်မယ် apache2 ကို install ပြီး service ကို restart ချချင်တာ ဒါကြောင့် service module ကိုသုံးတာဖြစ်တယ်။ service stateတွေမှာ restarted stopped started ဆိုပြီးရှိပါတယ်။ ဒါဆို playbook ကို run လိုက်ရအောင်။
```bash
root@master:~# ansible-playbook apache.yaml 

PLAY [Install Apache2] ********************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [10.28.212.239]

TASK [upate packages] ********************************************************************************************************************************
changed: [10.28.212.239]

TASK [install apache2] *******************************************************************************************************************************
ok: [10.28.212.239]

PLAY RECAP *******************************************************************************************************************************************
10.28.212.239              : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
node01 က webserver ထဲပါတယ် ။ ဒါကြောင့် node01 သွားပြီး systemctl နဲ့ apache2 ကိုစစ်ကြည့်ရအောင်။
```bash
root@node01:~# systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2021-07-09 06:04:01 UTC; 3s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 6638 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
   Main PID: 6642 (apache2)
      Tasks: 55 (limit: 19207)
     Memory: 18.0M
     CGroup: /system.slice/apache2.service
             ├─6642 /usr/sbin/apache2 -k start
             ├─6643 /usr/sbin/apache2 -k start
             └─6644 /usr/sbin/apache2 -k start

Jul 09 06:04:00 node01 systemd[1]: Starting The Apache HTTP Server...
Jul 09 06:04:01 node01 systemd[1]: Started The Apache HTTP Server.
```
apache2 က active နေပါပြီ။ ကဲ.. ဒီလောက်ဆို playbook ကို သဘောပေါက်လောက်ပြီထင်ပါတယ်။ playbook တွေများများ ရေးကြည့်ကြပါ။ PART-II ကတော့ ဒီလောက်ပါပဲခင်ဗျာ။ PART-III မှာ ansible-galaxy နဲ့ roles တွေအကြောင်း ရေးသွားပါမယ်။ bye!!

Thanks for reading ..
