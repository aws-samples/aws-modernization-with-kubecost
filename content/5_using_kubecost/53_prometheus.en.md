---
title : "Integrating Kubecost with AWS Managed Prometheus (AMP)"
chapter: false
weight : 53
---
### Overview

In the collaboration with Amazon Web Services (AWS), [Kubecost](https://www.kubecost.com/) integrates with [Amazon Managed Service for Prometheus (AMP)](https://docs.aws.amazon.com/prometheus/index.html) - a managed Prometheus-compatible monitoring service - to enable the customer to easily monitor Kubernetes cost at scale. In this module, you can learn how to integrate existing Kubecost installation with the new AMP instance.

### Reference Architecture diagram:

![kubecost-eks-amp](/images/AWS-AMP-integ-architecture.png)

### Instructions

#### Create Amazon Managed for Prometheus (AMP) instance:

Run the following command to get the information of your current EKS cluster:

```bash
kubectl config current-context
```
The example output should be in this format:

```bash
arn:aws:eks:${AWS_REGION}:${YOUR_AWS_ACCOUNT_ID}:cluster/${YOUR_CLUSTER_NAME}
```

Next, run the following command to create new a AMP intance

```bash
export AWS_REGION=<YOUR_AWS_REGION>
aws amp create-workspace --alias kubecost-amp --region $AWS_REGION
```

The AMP instance should be created in few seconds. Run the following command to get the workspace ID:

```bash
export AMP_WORKSPACE_ID=$(aws amp list-workspaces --region ${AWS_REGION} --output json --query 'workspaces[?alias==`kubecost-amp`].workspaceId | [0]' | cut -d'"' -f 2)
echo $AMP_WORKSPACE_ID
```

Take note of AMP_WORKSPACE_ID for using later

#### Set environment variables for integrating Kubecost with AMP

Run the following command to set environment variables for integrating Kubecost with AMP

```bash
export YOUR_CLUSTER_NAME=<YOUR_EKS_CLUSTER_NAME>
export YOUR_AWS_ACCOUNT_ID=<YOUR_AWS_ACCOUNT_ID>
export REMOTEWRITEURL="https://aps-workspaces.us-west-2.amazonaws.com/workspaces/${AMP_WORKSPACE_ID}/api/v1/remote_write"
export QUERYURL="http://localhost:8005/workspaces/${AMP_WORKSPACE_ID}"
```

#### Set up IRSA to allow Kubecost and Prometheus to read & write metrics from AMP

These following commands help to automate the following tasks:
- Create an IAM role with the AWS managed IAM policy and trusted policy for the following service accounts: `kubecost-cost-analyzer`, `kubecost-prometheus-server`.
- Modify current K8s service accounts with annotation to attach new IAM role.

```bash
eksctl create iamserviceaccount \
    --name kubecost-cost-analyzer \
    --namespace kubecost \
    --cluster ${YOUR_CLUSTER_NAME} --region ${AWS_REGION} \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonPrometheusQueryAccess \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonPrometheusRemoteWriteAccess \
    --override-existing-serviceaccounts \
    --approve
```

```bash
eksctl create iamserviceaccount \
    --name kubecost-prometheus-server \
    --namespace kubecost \
    --cluster ${YOUR_CLUSTER_NAME} --region ${AWS_REGION} \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonPrometheusQueryAccess \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonPrometheusRemoteWriteAccess \
    --override-existing-serviceaccounts \
    --approve
```

> **Note**: each command can take 2-3 minutes to complete.

For more information, you can check AWS documentation at [IAM roles for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) and learn more about AMP managed policy at [Identity-based policy examples for Amazon Managed Service for Prometheus](https://docs.aws.amazon.com/prometheus/latest/userguide/security_iam_id-based-policy-examples.html)

#### Integrating Kubecost with AMP

You can run this command to update Kubecost Helm release to use your AMP workspace as a time series database.

```bash
helm upgrade -i kubecost \
oci://public.ecr.aws/kubecost/cost-analyzer --version 1.97.0 \
--namespace kubecost --create-namespace \
-f https://tinyurl.com/kubecost-amazon-eks \
-f https://tinyurl.com/kubecost-amp \
--set global.amp.prometheusServerEndpoint=${QUERYURL} \
--set global.amp.remoteWriteService=${REMOTEWRITEURL}
```
#### Restarting Prom to apply new configuration

Run the following command to restart the Prometheus deployment to reload the service account configuration:

```bash
kubectl rollout restart deployment/kubecost-prometheus-server -n kubecost
```
Your Kubecost setup is now start writing and collecting data from AMP. Data should be ready for viewing within 15 minutes. For advanced configuration, you can learn more about this integration at [Kubecost documentation](https://guide.kubecost.com/hc/en-us/articles/4409859798679-Amazon-Managed-Service-for-Prometheus)
