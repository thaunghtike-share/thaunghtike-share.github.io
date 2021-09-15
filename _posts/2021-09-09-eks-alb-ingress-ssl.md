---
layout:     post
title:   "Setting Up TLS Encryption On Amazon EKS With ALB Ingress Controller"
date:       2021-09-15 15:13:18 +0200
image: 30.jpg
tags:
    - Kubernetes
    - EKS
categories: kubernetes
---   

<h2> Introduction </h2>

In this tutorial, I' gonna show you how to set up tls encryption on eks cluster with alb ingress controller. I already wrote a post about ssl nginx controller controller on my blog. You can read it [here](https://thaunghtike-share.github.io/2021/07/30/letencrypt-ssl-gke). In order for the Ingress resource to work, the cluster must have an ingress controller running. There are a lot of ingress controllers such as nginx, traefik and alb.

<h2> Prerequities </h2>

<ul>
    <li> eksctl </li>
    <li> eks cluster </li>
    <li> public hosted zone </li>
    <li> domain </li>
</ul>

<h2> Install Eksctl Binary </h2>

Before creating an eks cluster, we have to install eksctl binary to perform eks cluster operaions.

```bash
$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
$ sudo mv /tmp/eksctl /usr/local/bin
$ eksctl version
```
<h2> Create An EKS Cluster </h2>
 
Firstly we have to create an eks cluster named eksdemo1 in us-east-east-1. It takes 15-20 minutes depending on your connection. You can go to aws cloudformation stack and you can see what resources are deployed.
 
 ```bash
 $ eksctl create cluster --name=eksdemo1 --region=us-east-1 --zones=us-east-1a,us-east-1b --without-nodegroup
 ```
 <h2> Create & Associate IAM OIDC Provider for our EKS Cluster </h2>
 
 To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create & associate OIDC identity provider. To do so using eksctl we can use the below command. 
 
 ```bash
 $ eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster eksdemo1 \
    --approve
```    
Yeah you got an eks cluster. Let's create a node group with two worker nodes to join new created cluster named eksdemo1. It also takes 15-20 minutes to fully create a node group.
 
 ```bash
 $ eksctl create nodegroup --cluster=eksdemo1 --region=us-east-1  --name=eksdemo1-ng-public1 --node-type=t3.medium --nodes=2 --nodes-min=2                       --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=aws --managed --asg-access --external-dns-access --full-ecr-access                       --appmesh-access
 ```
 Now, you've created an eks cluster. Check with 'eksctl get clusters'. You will see one cluster is running. 
 
 $ 
 

<h2> Create A Kubernetes Service Account For Alb Ingress Controller</h2>

Then we have to create a k8s service account named alb-ingress-controller in kube-system namespace. This service account will help eks cluster to create and delete elastic load balancers.

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/master/docs/examples/rbac-role.yaml
$ kubectl get sa alb-ingress-controller -n kube-system -o yaml
```
<h2> Create IAM Policy for ALB Ingress Controller </h2>

This IAM policy will allow our ALB Ingress Controller pod to make calls to AWS APIs.

```bash
$ aws iam create-policy \
    --policy-name ALBIngressControllerIAMPolicy \
    --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/master/docs/examples/iam-policy.json
```

There is an issue in this policy. So we need to edit this policy. Go to Services -> IAM -> Policies. Click Visual Editor on this policy. Add ELB full access

    Click on Add Additional Permissions
        Service: ELB
        Actions: All ELB actions (elasticloadbalancing:*)
        Resources: All Resources




