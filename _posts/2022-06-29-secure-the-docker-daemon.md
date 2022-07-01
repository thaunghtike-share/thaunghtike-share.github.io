---
layout:     post
title:   "Secure The Docker Daemon"
date:       2022-06-28 15:13:18 +0200
image: dockertls.png
tags:
    - Docker
categories: Docker
---

ကျွန်တော် ဒီနေ့ မှာတော့ CKS ဖြေဖို့လေ့လာနေတုန်း သိလိုက်ရတဲ့ Secure The Docker Daemon ဆိုတဲ့ ခေါင်းစဥ်အောက်မှာနားလည်သလောက်ရှင်းပြပေးသွားမှာဖြစ်ပါတယ်။ Secure လုပ်တာကိုမဆွေးနွေးခင်မှာ Docker Architecture အကြောင်းကိုအနည်းငယ်ပြန်စလိုက်ရအောင်ဗျာ။

<h2> What is docker ? </h2>

Docker ဆိုတာ ကျွန်တော်တို့ရဲ့ application တွေကို containerized လုပ်ပြီး ship လုပ်ဖို့ run ဖို့အတွက် သုံံးတဲ့ platform တစ်ခုပါ။ ကျွန်တော်တို့ရဲ့ application တစ်ခုချင်းစီ run ဖို့အတွက် container လို့ခေါ်တဲ့ environment တစ်ခုကို create လုပ်ပေးသွားမှာဖြစ်တယ်။ 

<h2> Docker Architecture </h2>

Docker က client-server architecture ပုံစံအလုပ်လုပ်ပါတယ်။ Docker daemon က server-side ဖြစ်ပြီး users တွေခေါ်လိုက်တဲ့ docker client cli ကနေတစ်ဆင့် host ပေါ်မှာ containers တွေ image တွေ volumes တွေနဲ့ဆိုင်တဲ့ operations ကို manage လုပ်ပေးသွားပါတယ်။ Docker client က system တစ်ခုတည်းမှာရှိနေတဲ့ docke daemon ကို interact လုပ်နိုင်သလို remote system ပေါ်က docker daemon ကိုလည်း interact လုပ်နိုင်ပါတယ်။

![dact](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/dact.png)

Docker daemon နဲ့ client ဟာ REST API တွေကိုအသုံးပြုပြီး unix socket ကနေ communicate လုပ်ပါတယ်။ ပုံမှန်အားဖြင့် docker daemon ဟာ unix socket တစ်ခုကို /var/run/docker.sock မှာ create လုပ်ပေးပါတယ်။ အဲ့ဒီ socker ပေါ်မှာ client နဲ့ daemon က communicate လုပ်သွားတာပါ။ 

<h2> Acess the docker daemon remotely </h2>

ဒီနေရာမှာ client နဲ့ daemon ဟာ system တစ်ခုတည်းမှာဆို ပြသနာမရှိပါဘူး။ remote sytem က client တစ်ခုကနေ access လုပ်ဖို့လိုလာပြီဆို TCP socket တစ်ခုကို docker daemon မှာ create ပေးရပါမယ်။ Daemon ကို access ရသွားတဲ့သူတိုင်းဟာ container တွေကို ကြိုက်သလို manage လုပ်ဖို့ အခွင့်ရေးရသွားပါတယ်။ container တွေ volumes တွေကို ဖျက်နိုင်တယ်။ privileged container ကိုသုံးပြီး root system ကိုပါ access ရသွားမှာပါ။ 

ဒါကြောင့် TCP socket ကို create တဲ့အခါမှာ docker daemon ကို secure ဖြစ်ဖို့လိုအပ်လာပါတယ်။ secure ဖြစ်ဖို့ဆို tls encryption အတွက်လိုအပ်တဲ့ CA တွေ certificate တွေကို create ရပါမယ်။ tls encrypt သာမလုပ်ဘူးဆိုရင် docker daemon ရဲ့ host ip ကိုသီတဲ့ ဘယ်သူမဆို access ပေးသလိုဖြစ်သွားမှာပါ။ Docker ရဲ့ port က 2375 ဖြစ်ပြီး tls ကိုသုံးထားရင်တော့ 2376 ဖြစ်ပါတယ်။

<h2> Create CA, client and server certificates </h2>

ဒီအဆင့်မှာတော့ ကျွန်တော်တို့က tls encrypt ဖို့လိုအပ်တဲ့ CA နဲ့ server အတွက် certificates တွေကို create ပေးရပါမယ်။ CA ကနေ server အတွက် cert ကို sign ထိုးပေးမှာဖြစ်ပြီး client အတွက်လည်း cert ကိုလည်း အဲ့ဒီ CA ကနေပဲ sign ထိုးပေးရမှာဖြစ်ပါတယ်။ အရင်ဆုံး အောက်က script ကို run လိုက်ပါမယ်။ 

```bash
set -eu

#set -x ; debugging

cd ~
echo "you are now in $PWD"

if [ ! -d ".docker/" ] 
then
    echo "Directory ./docker/ does not exist"
    echo "Creating the directory"
    mkdir .docker
fi

cd .docker/
echo "type in your certificate password (characters are not echoed)"
read -p '>' -s PASSWORD

echo "Type in the server name you’ll use to connect to the Docker server"
read -p '>' SERVER

# 256bit AES (Advanced Encryption Standard) is the encryption cipher which is used for generating certificate authority (CA) with 2048-bit security.
openssl genrsa -aes256 -passout pass:$PASSWORD -out ca-key.pem 2048 

# Sign the the previously created CA key with your password and address for a period of one year.
# i.e. generating a self-signed certificate for CA
# X.509 is a standard that defines the format of public key certificates, with fixed size 256-bit (32-byte) hash
openssl req -new -x509 -days 365 -key ca-key.pem -passin pass:$PASSWORD -sha256 -out ca.pem -subj "/C=TR/ST=./L=./O=./CN=$SERVER"

# Generating a server key with 2048-bit security
openssl genrsa -out server-key.pem 2048

# Generating a certificate signing request (CSR) for the the server key with the name of your host.
openssl req -new -key server-key.pem -subj "/CN=$SERVER"  -out server.csr

echo subjectAltName = DNS:$SERVER,IP:$SERVER,IP:127.0.0.1 >> extfile.cnf
echo extendedKeyUsage = serverAuth >> extfile.cnf
# Sign the key with your password for a period of one year
# i.e. generating a self-signed certificate for the key
openssl x509 -req -days 365 -in server.csr -CA ca.pem -CAkey ca-key.pem -passin "pass:$PASSWORD" -CAcreateserial -out server-cert.pem -extfile extfile.cnf

# For client authentication, create a client key and certificate signing request
# Generate a client key with 2048-bit security
openssl genrsa -out key.pem 2048
# Process the key as a client key.
openssl req -subj '/CN=client' -new -key key.pem -out client.csr

# To make the key suitable for client authentication, create an extensions config file:
sh -c 'echo "extendedKeyUsage = clientAuth" > extfile.cnf'

# Sign the (public) key with your password for a period of one year
openssl x509 -req -days 365 -in client.csr -CA ca.pem -CAkey ca-key.pem -passin "pass:$PASSWORD" -CAcreateserial -out cert.pem -extfile extfile.cnf

echo "Removing unnecessary files i.e. client.csr extfile.cnf server.csr"
rm ca.srl client.csr extfile.cnf server.csr

echo "Changing the permissions to readonly by root for the server files."
# To make them only readable by you: 
chmod 0400 ca-key.pem key.pem server-key.pem

echo "Changing the permissions of the client files to read-only by everyone"
# Certificates can be world-readable, but you might want to remove write access to prevent accidental damage
# these are all x509 certificates aka public key certificates
# X.509 certificates are used in many Internet protocols, including TLS/SSL, which is the basis for HTTPS.
chmod 0444 ca.pem server-cert.pem cert.pem
```
CA certificate အတွက်လိုအပ်တဲ့ password နဲ့ server ကိုထည့်ပေးပါ။ ပြီးသွားရင်တော့ home directory ရဲ့ .docker ဆိုတဲ့ path ထဲမှာအောက်ကအတိုင်း တွေ့ရမှာဖြစ်ပါတယ်။

```bash
~ % ls .docker/
ca-key.pem  ca.pem  cert.pem  key.pem  server-cert.pem  server-key.pem
```
ပြီးသွားရင်တော့ docker service မှာ tls ကိုသုံးဖို့ update လုပ်ပေးရပါမယ်။ အရင်ဆုံး systemd configuration ကိုလုပ်ပေးပါမယ်။ directory တစ်ခုဆောက်လိုက်ပါ။

```bash
~ % sudo mkdir /etc/systemd/system/docker.service.d
```
ပြီးသွားရင်တော့ docker daemon မှာ tls encryption ကို configure အောက်ပါအတိုင်းလုပ်ပေးရပါမယ်။

```bash
~ % sudo vim /etc/systemd/system/docker.service.d/override.conf
```
အောက်က configuration ကို ထည့်လိုက်ပါ။

```bash
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -D -H unix:///var/run/docker.sock --tlsverify --tlscert=/home/ubuntu/.docker/server-cert.pem --tlscacert=/home/ubuntu/.docker/ca.pem --tlskey=/home/ubuntu/.docker/server-key.pem -H tcp://0.0.0.0:2376
```
အားလုံးပြီးသွားရင်တော့ systemctl ကို reload လုပ်ပေးလိုက်ပါ။

```bash
~ % sudo systemctl daemon-reload && sudo systemctl restart docker
```
<h2> Access the docker daemon remotely </h2>

ဒါဆိုရင်တော့ remote client ကနေ ခုဏက docker daemon ကို access လုပ်ကြည့်ရအောင်။ access လုပ်ဖို့အတွက် remote client မှာ ca.pem နဲ့ CA ကနေ sign ထိုးပေးခဲ့တဲ့ client အတွက်  private key နဲ့ cert ရှိရပါမယ်။ အောက်ကအတိုင်း access လုပ်နိုင်ပါတယ်။

```bash
~ % docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem -H=tcp://10.0.6.67:2376 ps -a
```
tls flag တွေကို docker commands တွေခေါ်တိုင်းမသုံးချင်ဘူးဆိုရင် env var တွေ export လုပ်ပြီးသုံးနိုင်ပါတယ်။

```bash
~ % export HOST=10.0.6.67
    export DOCKER_HOST=tcp://$HOST:2376 DOCKER_TLS_VERIFY=1
    docker ps -a
```

အဆုံးထိဖတ်ရှုပေးခဲ့ကြသူတိုင်းကိုကျေးဇူးတင်ပါတယ်။
