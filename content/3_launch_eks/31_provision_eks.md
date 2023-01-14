---
title: "Provision EKS Cluster"
chapter: false
weight: 31
---


{{% notice warning %}}
**DO NOT PROCEED** with this step unless you have [validated the IAM role](/2_setup/27_workspaceiam/#validate-the-iam-role) in use by the Cloud9 IDE. You will not be able to run the necessary kubectl commands in the later modules unless the EKS cluster is built using the IAM role.
{{% /notice %}}

#### Challenge:

**How do I check the IAM role on the workspace?**

{{%expand "Expand here to see the solution" %}}
Run `aws sts get-caller-identity` and validate that your _Arn_ contains `kubecost-workshop-admin`and an Instance Id.

**Example Output:**
```output
{
    "Account": "123456789012",
    "UserId": "AROA1SAMPLEAWSIAMROLE:i-01234567890abcdef",
    "Arn": "arn:aws:sts::123456789012:assumed-role/kubecost-workshop-admin/i-01234567890abcdef"
}
```

If you do not see the correct role, please go back and [validate the IAM role](/2_setup/27_workspaceiam/#validate-the-iam-role) for troubleshooting.

If you do see the correct role, proceed to next step to create an EKS cluster.
{{% /expand %}}

### Create an EKS cluster

{{% notice warning %}}
`eksctl` version must be 0.58.0 or above to deploy EKS 1.21, [click here](../212_prerequisites) to get the latest version.
{{% /notice %}}

Create an eksctl deployment file (kubecost-workshop.yaml) use in creating your cluster using the following syntax:

```bash
cat << EOF > kubecost-workshop.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: kubecost-workshop-eksctl
  region: ${AWS_REGION}
  version: "1.23"

availabilityZones: ["${AZS[0]}", "${AZS[1]}", "${AZS[2]}"]

managedNodeGroups:
- name: nodegroup
  desiredCapacity: 3
  instanceType: t3.large
  volumeSize: 80
  ssh:
    enableSsm: true

EOF
```

Next, use the file you created as the input for the eksctl cluster creation.

{{% notice info %}}
We are deliberatly launching at least one Kubernetes version behind the latest available.  Please review [Amazon EKS Kubernetes versions](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html) to determine what supported versions are currently available. 
{{% /notice %}}

```bash
eksctl create cluster -f kubecost-workshop.yaml
```

{{% notice info %}}
Launching EKS and all the dependencies will take approximately 15 minutes.
{{% /notice %}}

### Test cluster after completion:
Confirm your nodes:

```bash
kubectl get nodes # if we see our 3 nodes, we know we have authenticated correctly
```

### Update the kubeconfig file to interact with you cluster:
```bash
aws eks update-kubeconfig --name kubecost-workshop-eksctl --region ${AWS_REGION}
```


<!-- #### Export the Worker Role Name for use throughout the workshop:

```bash
STACK_NAME=$(eksctl get nodegroup --cluster kubecost-workshop-eksctl -o json | jq -r '.[].StackName')
ROLE_NAME=$(aws cloudformation describe-stack-resources --stack-name $STACK_NAME | jq -r '.StackResources[] | select(.ResourceType=="AWS::IAM::Role") | .PhysicalResourceId')
echo "export ROLE_NAME=${ROLE_NAME}" | tee -a ~/.bash_profile
``` -->

### Congratulations!

You now have a fully working Amazon EKS Cluster that is ready to use! Before you move on to any other labs, make sure to complete the steps on the next page to update the EKS Console Credentials.
