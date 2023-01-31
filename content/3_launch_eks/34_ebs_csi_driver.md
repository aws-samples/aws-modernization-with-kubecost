---
title: "Deploy Amazon EBS CSI driver"
chapter: false
weight: 34
---

### Deploy the Amazon EBS CSI Driver

{{% notice note %}}
The [Amazon Elastic Block Store (Amazon EBS) Container Storage Interface (CSI) driver](https://www.eksworkshop.com/beginner/170_statefulset/ebs_csi_driver/#:~:text=Amazon%20Elastic%20Block%20Store%20(Amazon%20EBS)%20Container%20Storage%20Interface%20(CSI)%20driver) provides a [CSI interface](https://www.eksworkshop.com/beginner/170_statefulset/ebs_csi_driver/#:~:text=The%20Container%20Storage%20Interface) that allows Amazon Elastic Kubernetes Service (Amazon EKS) clusters to manage the lifecycle of Amazon EBS volumes for persistent volumes. This step is required if your Amazon EKS cluster version is 1.23 and above.
{{% /notice %}}

#### Create a IAM role and ServiceAccount

```bash
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster ${CLUSTER_NAME} \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --override-existing-serviceaccounts \
    --approve \
    --role-only \
    --role-name AmazonEKS_EBS_CSI_DriverRole
export SERVICE_ACCOUNT_ROLE_ARN=$(aws iam get-role --role-name AmazonEKS_EBS_CSI_DriverRole | jq -r '.Role.Arn')
```

#### Deploy the Amazon EBS CSI Driver using Amazon EKS-addon

The Amazon EBS CSI Driver will deploy from the Amazon EKS-Addon

```bash
eksctl create addon --name aws-ebs-csi-driver --cluster $CLUSTER_NAME \
    --service-account-role-arn $SERVICE_ACCOUNT_ROLE_ARN --force
```