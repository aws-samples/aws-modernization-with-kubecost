---
title: "Undeploy the applications"
chapter: false
weight: 991
---


Undeploy the applications and clean up Identity resources:

```bash
helm delete kubecost -n kubecost

helm uninstall aws-load-balancer-controller \
    -n kube-system

kubectl delete -k github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master

aws iam delete-policy \
    --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy
```