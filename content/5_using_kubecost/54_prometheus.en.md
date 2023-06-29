---
title: "Integrating Kubecost with AWS Managed Prometheus (AMP)"
chapter: false
weight: 54
---
### Overview

In collaboration with AWS, Kubecost integrates with [Amazon Managed Service for Prometheus (AMP)](https://aws.amazon.com/prometheus/) - a managed Prometheus-compatible monitoring service - to enable customers to easily monitor Kubernetes cost at scale. In this module, you will learn how to integrate an existing Kubecost installation with AMP.

### Reference Architecture

![kubecost-eks-amp](/images/AWS-AMP-integ-architecture.png)

### Instructions

#### Create an AMP instance

Run the following command to create new a AMP instance.

```bash
aws amp create-workspace --alias kubecost-amp --region $AWS_REGION
```

The AMP instance should be created in a few seconds. Run the following command to get the workspace ID.

```bash
export AMP_WORKSPACE_ID=$(aws amp list-workspaces --region ${AWS_REGION} --output json --query 'workspaces[?alias==`kubecost-amp`].workspaceId | [0]' | cut -d'"' -f 2)
echo $AMP_WORKSPACE_ID
```

#### Set environment variables for integrating Kubecost with AMP

Run the following command to set environment variables for integrating Kubecost with AMP.

```bash
export CLUSTER_NAME=$(eksctl get clusters --region ${AWS_REGION} -o json | jq -r .[0].Name)
export REMOTEWRITEURL="https://aps-workspaces.${AWS_REGION}.amazonaws.com/workspaces/${AMP_WORKSPACE_ID}/api/v1/remote_write"
export QUERYURL="http://localhost:8005/workspaces/${AMP_WORKSPACE_ID}"
```

#### Set up IRSA to allow Kubecost and Prometheus to read and write metrics from AMP

These following commands help to automate the following tasks:

- Create an IAM role with the AWS managed IAM policy and trusted policy for the following service accounts: `kubecost-cost-analyzer`, `kubecost-prometheus-server`.
- Modify current Kubernetes service accounts with the annotation to attach a new IAM role.

```bash
eksctl create iamserviceaccount \
    --name kubecost-cost-analyzer \
    --namespace kubecost \
    --cluster ${CLUSTER_NAME} --region ${AWS_REGION} \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonPrometheusQueryAccess \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonPrometheusRemoteWriteAccess \
    --override-existing-serviceaccounts \
    --approve
```

```bash
eksctl create iamserviceaccount \
    --name kubecost-prometheus-server \
    --namespace kubecost \
    --cluster ${CLUSTER_NAME} --region ${AWS_REGION} \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonPrometheusQueryAccess \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonPrometheusRemoteWriteAccess \
    --override-existing-serviceaccounts \
    --approve
```

> **Note**: each command can take 2-3 minutes to complete.

For more information, you can check AWS documentation at [IAM roles for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) and learn more about AMP managed policy at [Identity-based policy examples for Amazon Managed Service for Prometheus](https://docs.aws.amazon.com/prometheus/latest/userguide/security_iam_id-based-policy-examples.html)

#### Integrating Kubecost with AMP

You can run this command to update the Kubecost Helm release to use your AMP workspace as a time series database.

```bash
helm upgrade -i kubecost \
oci://public.ecr.aws/kubecost/cost-analyzer --version="$VERSION" \
--namespace kubecost --create-namespace \
-f https://tinyurl.com/kubecost-amazon-eks \
-f https://tinyurl.com/kubecost-amp \
--set global.amp.prometheusServerEndpoint=${QUERYURL} \
--set global.amp.remoteWriteService=${REMOTEWRITEURL}
```

#### Restarting Prometheus to apply the new configuration

Run the following command to restart the Prometheus deployment to reload the service account configuration:

```bash
kubectl rollout restart deployment/kubecost-prometheus-server -n kubecost
```

Your Kubecost setup will now begin writing and collecting data from AMP. Data should be ready for viewing within 15 minutes. For advanced configurations, you can learn more about this integration in the [Kubecost documentation](https://docs.kubecost.com/install-and-configure/install/custom-prom/aws-amp-integration).
