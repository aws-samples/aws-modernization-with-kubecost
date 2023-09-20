---
title: "Deploying Kubecost on an Amazon EKS cluster"
chapter: false
weight: 41
---

### Overview

Kubecost provides real-time cost visibility and insights for teams using Kubernetes, helping you continuously monitor the cost of the container workloads. In this section, you will learn how to deploy Kubecost on Amazon EKS cluster in few minutes with bundled Prometheus running locally.

### Reference Architecture

![kubecost-eks](/images/AWS-EKS-cost-monitoring-architecture.png)

### Instructions

#### Step 1: Install Kubecost on your Amazon EKS cluster

In your environment, run the following command from your terminal to install Kubecost on your existing Amazon EKS cluster:

```bash
export VERSION="1.106.0"
helm upgrade -i kubecost \
oci://public.ecr.aws/kubecost/cost-analyzer --version="$VERSION" \
--namespace kubecost --create-namespace \
-f https://raw.githubusercontent.com/kubecost/cost-analyzer-helm-chart/develop/cost-analyzer/values-eks-cost-monitoring.yaml \
--set prometheus.configmapReload.prometheus.enabled="false"
```

> **Note**: Remember to replace $VERSION with an actual version number. You can find all available versions [here](https://gallery.ecr.aws/kubecost/cost-analyzer).

Kubecost should be up and running in a few minutes. You can verify the installation progress using the following command:

```bash
kubectl get pod -n kubecost
```

Example response with all pods in a running state:

```bash
kubectl get pod -n kubecost
NAME                                          READY   STATUS    RESTARTS   AGE
kubecost-cost-analyzer-5c4b489cfd-kt5w2       2/2     Running   0          3m5s
kubecost-kube-state-metrics-99bb8c55b-454x6   1/1     Running   0          3m5s
kubecost-prometheus-server-f99987f55-q9b8d    1/1     Running   0          3m5s
```

#### Step 2: Generate Kubecost dashboard endpoint

Run the following command to enable port-forwarding to expose the Kubecost dashboard to your workspace:

```bash
kubectl port-forward --namespace kubecost deployment/kubecost-cost-analyzer 8080:9090
```

#### Step 3: Access cost monitoring dashboard

On your workspace console, navigate to `Preview -> Preview running application`. New windows appears on your workspace allowing you to access the Kubecost dashboard as in the following example screenshot:

![kubecost-ui-workspace](/images/kubecost-expose-ui-workspace.png)

You can now start monitoring your Amazon EKS cluster cost and efficiency. Depending on your organization's requirements and set up, you may have different options to expose Kubecost for internal access. You will learn about how to expose Kubecost using an AWS Application Load Balancer in the next module of this workshop.

Example Kubecost's overview page:

![kubecost-overview-example](/images/overview-example.png)
