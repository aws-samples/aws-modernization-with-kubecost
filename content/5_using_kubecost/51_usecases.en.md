---
title : "Use Cases for Kubecost"
chapter: false
weight : 51
---

# Overview

Kubecost has 3 primary use cases:

1. Gain visibility into what is consuming Kubernetes resources- and driving the costs.
2. Understand if those resources are being use efficiently.
3. Identify areas to save money.

Kubecost is an API first solution - meaning that everything available in the UI is also available via a API endpoint.

This lab will focus on the UI.

The main overview page show high-level trends and a summary of what this instance of Kubecost is managing.

![overview](/images/kubecost-overview.png)

To understand which resources are consuming the most resources in the monitored clusters, click Allocations in the right navigation tree.

In this view we can aggregate the resources by a variety of ways:
 - Kubernetes constructs: cluster, namespace, node, controllers, services, pod, label
 - Logical constructs mapped to labels: Environment, Department, Owner, Team

![aggregations](/images/kubecost-aggregations.png)

This view can be customized based on given requirements. Some of the most useful settings:
 - Idle Costs: idle costs are unused node capacity. Sharing by node will distribute this cost to the resources using that node. This is useful in a 100% chargeback model
 - Chart: Change the display based to Cost over time, Efficiency over time and other options

![allocation-settings](/images/kubecost-report-settings.png)

Within the UI, note that you can click the blue link to open a detailed view of that resources, or click the row to drill into that resource.

![allocation-drill-in](/images/kubecost-drill-in.png)
