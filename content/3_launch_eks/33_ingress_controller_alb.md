---
title: "Deploy Ingress Controller"
chapter: false
weight: 33
---

### Deploy the AWS Load Balancer Controller

{{% notice note %}}
Learn more about [Application load balancing on Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html) in the Amazon EKS documentation.
{{% /notice %}}


#### Prerequisites

We will first set our cluster name in an environment variable:

```bash
export CLUSTER_NAME=$(eksctl get clusters --region ${AWS_REGION} -o json | jq -r .[0].Name)
```

We will then verify if the AWS Load Balancer Controller version has been set

```bash
if [ ! -x ${LBC_VERSION} ]
  then
    tput setaf 2; echo '${LBC_VERSION} has been set.'
  else
    tput setaf 1;echo '${LBC_VERSION} has NOT been set.'
fi
```

If the result is <span style="color:red">${LBC_VERSION} has NOT been set.</span>, click [here](/2_setup/24_clistools.html#set-the-aws-load-balancer-controller-version) for the instructions or run the following command:

```bash
echo 'export LBC_VERSION="v2.4.1"' >>  ~/.bash_profile
echo 'export LBC_CHART_VERSION="1.4.1"' >>  ~/.bash_profile
.  ~/.bash_profile
```

We will use **Helm** to install the ALB Ingress Controller.

Check to see if `helm` is installed:

```bash
helm version --short
```

{{% notice info %}}
If `Helm` is not found, click [installing Helm CLI](/2_setup/24_k8stools) for instructions.
{{% /notice %}}

#### Create IAM OIDC provider

```bash
eksctl utils associate-iam-oidc-provider \
    --region ${AWS_REGION} \
    --cluster ${CLUSTER_NAME} \
    --approve
```

{{% notice note %}}
Learn more about [IAM Roles for Service Accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) in the Amazon EKS documentation.
{{% /notice %}}

#### Create an IAM policy

Create a policy called **AWSLoadBalancerControllerIAMPolicy**

```bash
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/${LBC_VERSION}/docs/install/iam_policy.json
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

#### Create a IAM role and ServiceAccount

```bash
eksctl create iamserviceaccount \
  --cluster ${CLUSTER_NAME} \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve
```

#### Install the TargetGroupBinding CRDs

```bash
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master"

kubectl get crd
```

#### Deploy the Helm chart

The helm chart will deploy from the eks repo

```bash
helm repo add eks https://aws.github.io/eks-charts

helm upgrade -i aws-load-balancer-controller \
    eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=${CLUSTER_NAME} \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set image.tag="${LBC_VERSION}" \
    --version="${LBC_CHART_VERSION}"

kubectl -n kube-system rollout status deployment aws-load-balancer-controller
```
