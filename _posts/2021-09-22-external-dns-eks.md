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
      serviceAccountName: external-dns
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
