---
title: "Policy and cost"
chapter: false
weight: 53
---

### Overview

Kubecost helps gain visibility into the costs and their drivers allowing you to make informed financial and operational decisions. However, the ability to continually operate Kubecost in a manner consistent with your organization's goals and requirements may necessitate additional guardrails. Developers and other users of the Kubecost-managed cluster should not be able to do things which cause visibility/organizational problems or overrun spend.

In this section, we will layer in policy to function as those guardrails to ensure that the proper governance is enforced.

### Prerequisites

- You have completed earlier modules in this course and have a running EKS cluster with Kubecost installed.

### Step 1: Deploy Kyverno

[Kyverno](https://kyverno.io) is a policy engine designed specifically for Kubernetes. Using Kyverno, policies are written as standard YAML and re-use other Kubernetes-native concepts and skills allowing you to implement policy quickly and easily. Kyverno runs as an admission controller in the cluster and allows resources which match installed policies to be validated, resulting in acceptance or denial, and mutated, resulting in resources being modified prior to creation.

Add the Kyverno repository to Helm.

```sh
helm repo add kyverno https://kyverno.github.io/kyverno/
```

Scan for new charts.

```sh
helm repo update
```

Install the Kyverno chart. This will deploy Kyverno with a single replica for each of its controllers.

> Note: When deploying Kyverno in production on EKS, some [additional steps](https://kyverno.io/docs/installation/platform-notes/#notes-for-eks-users) are recommended.

```sh
helm install kyverno kyverno/kyverno -n kyverno --create-namespace
```

Once Kyverno has been successfully installed, wait for around 30 seconds and check the status of all Pods. You should see they all show as being in a Running state.

```sh
kubectl -n kyverno get po
```

### Step 2: Create a policy to require Kubecost labels

As mentioned in the [use cases section](/5_using_kubecost/51_usecases.html), Kubecost has the ability to group Pods and their controllers by certain Kubernetes labels so you can view cost allocations on a more custom and granular basis. For example, in addition to cluster and namespace grouping, you may wish to group workloads by the department which created and is therefore responsible for them in order to determine which cost center is associated with those specific workloads. In order for Kubecost to perform this grouping, the `department` label must be assigned to Pods. If the label is not present, Kubecost does not know the given Pods are associated with a department and will therefore associate them with the "Unallocated workloads" group. It is therefore important that some governance be applied in a cluster to ensure the requisite labels are always present.

> Note: Kubecost allows for customization of these labels, but in this workshop we will use the defaults.

In order to guarantee this label assignment, we will use Kyverno to require that Pods have the `department` label by using the below policy. Inspect the policy and read the comments to understand what each portion does.

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy                        ### A ClusterPolicy operates across the entire cluster.
metadata:
  name: require-kubecost-labels            ### The policy name. A policy is a collection of rules.
spec:
  validationFailureAction: Enforce         ### The `Enforce` action means resources which are in violation will not be created.
  background: true                         ### Background mode means Kyverno will periodically scan the cluster to check for violations.
  rules:
  - name: require-labels                   ### The rule name. A policy may contain many rules where each rule is of a certain type.
    match:
      any:
      - resources:
          kinds:
          - Pod                            ### This rule matches on the Pod resource kind.
    validate:                              ### This is a `validate` rule meaning Kyverno will deliver a "yes" or "no" response.
      message: >-                          ### This message will be returned for Pods which violate the rule.
        "The Kubecost label `department`
        is required for Pods."
      pattern:                             ### The `pattern` field is a portion of the resource which should be matched.
        metadata:
          labels:
            department: "?*"               ### The `department` label is required to have some value, denoted by the `?*` value.
```

Create a YAML manifest named `kubecost-labels.yaml` and copy-and-paste the above policy into the file.

Prior to installation of the policy, let's first run one simple Pod which does not have the required label so we may see how Kyverno responds in a later step.

Create the following busybox Pod in your cluster and, afterwards, ensure it is present and in a running state.

```sh
kubectl run busybox --image busybox:latest -- sleep 1d
```

Now, install the policy.

```sh
kubectl create -f kubecost-labels.yaml
```

After the policy is created, check its status.

```sh
kubectl get clusterpolicy -o wide
```

You should see output similar to the below. Notice that the policy is in a ready state.

```sh
NAME                      BACKGROUND   VALIDATE ACTION   FAILURE POLICY   READY   AGE   VALIDATE   MUTATE   GENERATE   VERIFYIMAGES   MESSAGE
require-kubecost-labels   true         Enforce                            True    21s   1          0        0          0              Ready
```

### Step 3: See a validation policy in action

Now that the Kyverno policy has been created, attempt to create a new Pod in the cluster. Run the following command.

```sh
kubectl run nginx --image nginx:latest
```

Notice how the attempt to run a Pod this time was blocked with the following message.

```sh
Error from server: admission webhook "validate.kyverno.svc-fail" denied the request: 

resource Pod/default/nginx was blocked due to the following policies 

require-kubecost-labels:
  require-labels: 'validation error: "The Kubecost label `department` is required
    for Pods.". rule require-labels failed at path /metadata/labels/department/'
```

Kyverno has detected your attempt to create the Pod and saw that it did not contain the `department` label as required in the policy. It responded with a message indicating why the Pod was blocked.

Create the Pod once more but include the `department` label with some value. This should be allowed by Kyverno and the Pod will be created.

```sh
kubectl run nginx --image nginx:latest -l department=finance
```

Assuming this and the earlier busybox Pod was created in the `default` namespace, get all running Pods along with their labels.

```sh
kubectl get po --show-labels
```

You should see output similar to the below.

```sh
NAME      READY   STATUS    RESTARTS   AGE   LABELS
busybox   1/1     Running   0          21m   run=busybox
nginx     1/1     Running   0          26s   department=finance
```

Remember that the earlier busybox Pod was created prior to installation of the Kyverno policy and it did not contain the `department` label. The nginx Pod, created after introduction of the policy, did have the label. Why then is the busybox Pod still present?

### Step 4: Policy Reports

Kyverno does not cause disruption when a validation policy is introduced into a running cluster but will still scan and allow you to see preexisting resources which may be in violation of those policies. Kyverno will not touch the resources but will inform you of their disposition in a Policy Report.

A [Policy Report](https://kyverno.io/docs/policy-reports/) is an open standard, developed and maintained by the Kubernetes policy working group, which shows results of a policy on resources in a cluster. Kyverno creates and maintains Policy Reports as part of the policy lifecycle process.

Let's retrieve the policy reports in this namespace.

```sh
kubectl get policyreport
```

You should see something like the following.

```sh
NAME                           PASS   FAIL   WARN   ERROR   SKIP   AGE
cpol-require-kubecost-labels   1      1      0      0       0      27m
```

A policy report is created for every policy installed. The name of this policy report is `cpol-require-kubecost-labels` which references the type of policy (a ClusterPolicy, or "cpol" for short), and the name of the policy itself. Notice how there is one pass and one fail result.

Inspect the report to see the detailed results.

```sh
kubectl get policyreport cpol-require-kubecost-labels -o yaml
```

The full policy report is shown with contents similar to what is below.

```yaml
apiVersion: wgpolicyk8s.io/v1alpha2
kind: PolicyReport
metadata:
  creationTimestamp: "2023-06-27T12:37:29Z"
  generation: 9
  labels:
    app.kubernetes.io/managed-by: kyverno
    cpol.kyverno.io/require-kubecost-labels: "6733157"
  name: cpol-require-kubecost-labels
  namespace: default
  resourceVersion: "6741247"
  uid: 0d43d8fd-d98c-448b-9208-b030037fbe13
results:
- message: validation rule 'require-labels' passed.
  policy: require-kubecost-labels
  resources:
  - apiVersion: v1
    kind: Pod
    name: nginx
    namespace: default
    uid: de4c7723-b647-442a-b16f-e18621e3cd26
  result: pass
  rule: require-labels
  scored: true
  source: kyverno
  timestamp:
    nanos: 0
    seconds: 1687871908
- message: 'validation error: "The Kubecost label `department` is required for Pods.".
    rule require-labels failed at path /metadata/labels/department/'
  policy: require-kubecost-labels
  resources:
  - apiVersion: v1
    kind: Pod
    name: busybox
    namespace: default
    uid: f3348a9f-0705-401a-9a33-4756f9fe3c2d
  result: fail
  rule: require-labels
  scored: true
  source: kyverno
  timestamp:
    nanos: 0
    seconds: 1687871850
summary:
  error: 0
  fail: 1
  pass: 1
  skip: 0
  warn: 0
```

In the policy report, observe how the busybox Pod you created earlier is listed as having failed the policy while the nginx Pod passed.

Using policy reports created by Kyverno can be a powerful way to assist in developer self-service and ensure Kubecost is able to organize and display allocated costs effectively. Because policy reports are a standard Kubernetes custom resource, they can be queried and scraped by tools of your choosing, including tools like the open source [Policy Reporter](https://github.com/kyverno/policy-reporter). They can also be controlled with Kubernetes RBAC allowing namespace owners and users to view policy reports so they know what needs remediation.

### Cleanup

Set the policy to `Audit` mode for now which will allow Kyverno to permit all resources regardless of their labels. As you finish this workshop, you may then choose to inspect the policy reports generated by Kyverno.

```sh
kubectl patch clusterpolicy require-kubecost-labels --type merge -p '{"spec":{"validationFailureAction":"Audit"}}'
```
