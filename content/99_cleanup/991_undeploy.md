---
title: "Uninstallation"
chapter: false
weight: 1
---

Remove the components installed during this workshop:

1. Kubecost

```bash
helm delete kubecost -n kubecost
```

2. AWS Load Balancer Controller

```bash
helm uninstall aws-load-balancer-controller -n kube-system

kubectl delete -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master"

eksctl delete iamserviceaccount --cluster $CLUSTER_NAME --name aws-load-balancer-controller --namespace kube-system

aws iam delete-policy --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy
```

3. AWS EBS CSI Driver

```bash
eksctl delete addon --name aws-ebs-csi-driver --cluster $CLUSTER_NAME

eksctl delete iamserviceaccount --cluster $CLUSTER_NAME --name ebs-csi-controller-sa --namespace kube-system
```
