---
layout:     post
title:   "Setup And Use External DNS With Route 53 In Your EKS Cluster"
date:       2021-09-22 15:13:18 +0200
image: 31.jpg
tags:
    - Kubernetes
    - EKS
categories: kubernetes
---

<h2> What is External DNS? </h2>

Hi guys, I released a post about ssl termination on alb ingress controller last week. You can read it [albsslingress](https://thaunghtike-share.github.io/2021/09/15/eks-alb-ingress-ssl). You have to create a route 53 record with alb ingress dns name manually once you deployed ssl termination on ALB ingress. EKS supports you to create those records automatically when you create an ingress route or service. In short, external DNS is a pod running in your EKS cluster which watches over all your ingresses. When it detects an ingress with a host specified, it automatically picks up the hostname as well as the endpoint and creates a record for that resource in Route53. If the host is changed or deleted, external DNS will reflect the change immediately in Route53.

<h2> Create IAM policy </h2>

You have to create an IAM policy named AllowExternalDNSUpdates. This IAM policy will allow external-dns pod to add, remove DNS entries (Record Sets in a Hosted Zone) in AWS Route53 service. Go to Services -> IAM -> Policies -> Create Policy. Click on JSON Tab and copy paste below JSON. Then click on 'create policy'.

```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": [
        "arn:aws:route53:::hostedzone/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones",
        "route53:ListResourceRecordSets"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```
<h2> Create IAM Role, k8s Service Account & Associate IAM Policy </h2>

As part of this step, we are going to create a k8s Service Account named external-dns and also IAM role and associate them by annotating role ARN in Service Account. Create this service account under kube-system namespace. Replace policy arn with your policy arn.

```bash
# Replaced name, namespace, cluster, arn 
eksctl create iamserviceaccount \
    --name external-dns-demo \
    --namespace default \
    --cluster eksdemo1 \
    --attach-policy-arn arn:aws:iam::993450297386:policy/AllowExternalDNSUpdates \
    --approve \
    --override-existing-serviceaccounts
```    
Verify external-dns service account, primarily verify annotation related to IAM Role

```bash
$ kubectl get sa external-dns-demo -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::993450297386:role/eksctl-eksdemo1-addon-iamserviceaccount-defa-Role1-1VUEQBDOYDMCE
  creationTimestamp: "2021-09-22T16:00:12Z"
  labels:
    app.kubernetes.io/managed-by: eksctl
  name: external-dns-demo
  namespace: default
  resourceVersion: "51192"
  uid: 1d8eb723-463e-469e-84d2-6f8c3fe388ce
secrets:
- name: external-dns-demo-token-wvnkv
```
<h2> Update External DNS Kubernetes manifest </h2>

Before creating external dns deployment, you have to edit manifest a litte bit.

<ul>
    <li> Copy the role-arn you have made a note at the end of step-03 and replace at line no 9 </li>
    <li> Commented line 55 and 56. We used eksctl to create IAM role and attached the AllowExternalDNSUpdates policy. We didnt use KIAM or Kube2IAM so we don't need these two lines, so commented </li>
    <li> Commented line 65 and 67.Make sure external dns see all hosted zones in route 53. </li>
</ul>    

```bash
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: external-dns
rules:
- apiGroups: [""]
  resources: ["services","endpoints","pods"]
  verbs: ["get","watch","list"]
- apiGroups: ["extensions"]
  resources: ["ingresses"]
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list","watch"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: external-dns-viewer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-dns
subjects:
- kind: ServiceAccount
  name: external-dns-demo
  namespace: default
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: external-dns
  template:
    metadata:
      labels:
        app: external-dns
      # If you're using kiam or kube2iam, specify the following annotation.
      # Otherwise, you may safely omit it.  #Change-2: Commented line 55 and 56
      #annotations:  
        #iam.amazonaws.com/role: arn:aws:iam::ACCOUNT-ID:role/IAM-SERVICE-ROLE-NAME    
    spec:
      serviceAccountName: external-dns-demo
      containers:
      - name: external-dns
        image: registry.opensource.zalan.do/teapot/external-dns:latest
        args:
        - --source=service
        - --source=ingress
        # Change-3: Commented line 65 and 67 - --domain-filter=external-dns-test.my-org.com # will make ExternalDNS see only the hosted zones matching provided domain, omit to process all available hosted zones
        - --provider=aws
       # Change-3: Commented line 65 and 67  - --policy=upsert-only # would prevent ExternalDNS from deleting any records, omit to enable full synchronization
        - --aws-zone-type=public # only look at public hosted zones (valid values are public, private or no value for both)
        - --registry=txt
        - --txt-owner-id=my-hostedzone-identifier
      securityContext:
        fsGroup: 65534 # For ExternalDNS to be able to read Kubernetes and AWS token files
```        
Verify external dns pod is running.

```bash
$ kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/external-dns-75b6bbbb7b-r4hdr   1/1     Running   0          113s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   6h20m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/external-dns   1/1     1            1           116s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/external-dns-75b6bbbb7b   1         1         1       117s
```
Check the logs of the pod.

```bash
$ kubectl logs -f $(kubectl get po | egrep -o 'external-dns[A-Za-z0-9-]+')
time="2021-09-22T16:06:06Z" level=info msg="Instantiating new Kubernetes client"
time="2021-09-22T16:06:06Z" level=info msg="Using inCluster-config based on serviceaccount-token"
time="2021-09-22T16:06:06Z" level=info msg="Created Kubernetes client https://10.100.0.1:443"
time="2021-09-22T16:06:14Z" level=info msg="All records are already up to date"
time="2021-09-22T16:07:14Z" level=info msg="All records are already up to date"
time="2021-09-22T16:08:14Z" level=info msg="All records are already up to date"
```
<h2> Create Nginx Deployment </h2>

You are ready to create ingress routes on this eks cluster. Let’s create a sample nginx deployment for this demo using kubectl.

```bash
$ kubectl create deploy nginx --image nginx --port 80
```
make sure nginx deployment is ready.

```bash
$ kubectl get deploy 
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           63s
```
<h2>  Expose Nginx Service As NodePort  </h2>

This step is quite important. You have to expose nginx svc as NodePort. So, alb can create a listener which will forward to target group with this NodePort.

```bash
$ kubectl expose deploy nginx --type NodePort --target-port 80 --port 80
```
<h2> AWS ALB Ingress Controller - SSL  </h2>

Firstly you have to create a public hosted zone in Route53. I already created a public hosted zone named thaunghtikeoo.info.

![thoroute53](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/albhz.png)

<h2> Create a SSL Certificate in Certificate Manager </h2>

I will use amazon cert manager to request a certificate. Then select a dns validation method to validate your certificate with your domain. 

![certmanpend](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/albcp.png)

You see certificate is issued by ACM after some minutes.

![certissued](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/albci.png)

<h2> Create Ingress Manifest </h2>

This ingress rule will create an ALB on which ssl is already installed. You have to annotate ACM arn to install ssl on this ingress route. It will automatically create a dns record in route 53 hosted zone for nginx.thaunghtikeoo.info.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingresss-demo
  labels:
    app: nginx
  annotations:
    # Ingress Core Settings  
    kubernetes.io/ingress.class: "alb"
    alb.ingress.kubernetes.io/scheme: internet-facing
    # Health Check Settings
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP 
    alb.ingress.kubernetes.io/healthcheck-port: traffic-port
    #Important Note:  Need to add health check path annotations in service level if we are planning to use multiple targets in a load balancer    
    #alb.ingress.kubernetes.io/healthcheck-path: /
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: '15'
    alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '5'
    alb.ingress.kubernetes.io/success-codes: '200'
    alb.ingress.kubernetes.io/healthy-threshold-count: '2'
    alb.ingress.kubernetes.io/unhealthy-threshold-count: '2'
    ## SSL Settings
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}, {"HTTP":80}]'
    alb.ingress.kubernetes.io/certificate-arn:  arn:aws:acm:us-east-1:993450297386:certificate/b8a072e5-096b-436d-87da-f1517c72023d
    #alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-TLS-1-1-2017-01 #Optional (Picks default if not used)    
    # SSL Redirect Setting
    alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'   
    # External DNS - For creating a Record Set in Route53
    external-dns.alpha.kubernetes.io/hostname: nginx.thaunghtikeoo.info     
spec:
  rules:
    - http:
        paths:
          - path: /* # SSL Redirect Setting
            backend:
              serviceName: ssl-redirect
              servicePort: use-annotation                  
          - path: /*
            backend:
              serviceName: nginx
              servicePort: 80             
````
Create ssl ingress with kubectl. You got an ingress route running after some minutes.

```bash
 kubectl get ingress
NAME                  CLASS    HOSTS   ADDRESS                                                                  PORTS   AGE
nginx-ingresss-demo   <none>   *       6ad0ef5b-default-nginxingr-ce56-1065941475.us-east-1.elb.amazonaws.com   80      2m40s
```
Go to aws ec2 console and you see an alb with two listers is running.

![albpart2](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/albpart2.png)

Also go to route 53 hosted zone. You will see dns record are automatically created by external dns.

![nginxr53](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/nginxr53.png)

Access nginx.thaunghtikeoo.info on browser. You will see 'http to https redirect' also works because I also annotates this redirect feauture on ingress rule.

![nginxalbpage](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/nginxalbpage.png)

Also check the certificate detail. verify certificate is issued by amazon.

![ssldemocert](https://raw.githubusercontent.com/thaunghtike-share/thaunghtike-share.github.io/master/images/ssldemocert.png)

