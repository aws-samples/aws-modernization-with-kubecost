---
title : "The Kubecost API"
chapter: false
weight : 52
---

### Overview

Kubecost offers a number of API endpoints to query cost metrics data. The Kubecost API is especially useful in cases where you want to integrate Kubecost data with your existing services, trigger alerts, or set up automation pipelines.

### Prerequisites

- You have completed Modules 1 & 2 of this workshop
- You have an API client installed such as [Insomnia](https://insomnia.rest/), [Postman](https://www.postman.com/), or a CLI utility such as [HTTPie](https://httpie.io/cli) or [cURL](https://curl.se/).

### Step 1: Enable access to the Kubecost API

The Kubecost API can be queried using the `<KUBECOST_IP>:9090/model/` prefix. In order to do this, you'll either need to expose Kubecost locally, or via an AWS Application Load Balancer (ALB). Full steps for both are described in the [Deploying and Accessing Kubecost](/4_deploy_kubecost.html) module of this lab.

For the sake of simplicity, we'll access the API from a `curl` pod by running following command in our AWS Cloud9 environment terminal:

```bash
kubectl run curlpod --image=curlimages/curl -i --tty -- sh
ENDPOINT="http://kubecost-cost-analyzer.kubecost.svc.cluster.local:9090"
```

### Step 2: Explore the data

Here are some examples of data available in the Free Tier of the API:

#### [`/allocation`](https://docs.kubecost.com/apis/apis-overview/allocation)

The Kubecost Allocation API is used by the Kubecost Allocation frontend and retrieves cost allocation information for any Kubernetes concept, e.g. cost by namespace, label, deployment, service, and more. This API is directly integrated with the Kubecost ETL caching layer and CSV pipeline so it can scale to large clusters.

**Example 1**: Request allocation data for each 24-hour period in the last three days, aggregated by namespace by running following command in your AWS Cloud9 environment terminal:

```bash
curl $ENDPOINT/model/allocation \
  -d window=3d \
  -d aggregate=namespace \
  -d accumulate=false \
  -d shareIdle=false \
  -G
```

Note: querying for "3d" will likely return a range of four sets because the queried range will overlap with four precomputed 24-hour sets, each aligned to the configured timezone.

**Example 2**:

Allocation data for the last one day, multi-aggregated by namespace and deployment, and accumulated into one allocation for the entire window.

```bash
curl $ENDPOINT/model/allocation \
  -d window=1d \
  -d aggregate=namespace,deployment \
  -d accumulate=true \
  -G
```

#### [`/assets`](https://docs.kubecost.com/apis/apis-overview/assets-api)

Assets API retrieves the backing cost data broken down by individual assets, e.g. node, disk, etc, and provides various aggregations of this data. Optionally provides the ability to integrate with external cloud assets.

API parameters include the following:

- `window` dictates the applicable window for measuring historical asset cost. Currently, supported options are as follows:
  - "15m", "24h", "7d", "48h", etc.
  - "today", "yesterday", "week", "month", "lastweek", "lastmonth"
  - "1586822400,1586908800", etc. (start and end unix timestamps)
  - "2020-04-01T00:00:00Z,2020-04-03T00:00:00Z", etc. (start and end UTC RFC3339 pairs)
- `aggregate` is used to consolidate cost model data. Supported aggregation types are cluster and type. Passing an empty value for this parameter, or not passing one at all, returns data by an individual asset.
- `accumulate` when set to false this endpoint returns daily time series data vs cumulative data. Default value is false.
- `disableAdjustments` when set to true, zeros out all adjustments from cloud provider reconciliation, which would otherwise change the totalCost.
- `format` when set to `csv`, will download an accumulated version of the asset results in CSV format. By default, results will be in JSON format.

This API returns a set of JSON objects in this format of following example:

```bash
curl $ENDPOINT/model/assets \
  -d window=today \
  -d aggregate=type \
  -d accumulate=true \
  -d disableAdjustments=true \
  -d format=json \
  -G
```

Example results:

```json
{
  "code": 200,
  "data": [
    {
      "ClusterManagement": {
        "type": "ClusterManagement",
        "properties": {
          "category": "Management",
          "provider": "AWS",
          "service": "Kubernetes"
        },
        "labels": {},
        "window": {
          "start": "2022-11-07T00:00:00Z",
          "end": "2022-11-08T00:00:00Z"
        },
        "start": "2022-11-07T00:00:00Z",
        "end": "2022-11-08T00:00:00Z",
        "minutes": 1440,
        "totalCost": 4.056748
      },
      "Disk": {
        "type": "Disk",
        "properties": {
          "category": "Storage",
          "provider": "AWS",
          "service": "Kubernetes"
        },
        "labels": {},
        "window": {
          "start": "2022-11-07T00:00:00Z",
          "end": "2022-11-08T00:00:00Z"
        },
        "start": "2022-11-07T00:00:00Z",
        "end": "2022-11-07T20:22:00Z",
        "minutes": 1222,
        "byteHours": 14907798980812.8,
        "bytes": 731970490056.2749,
        "byteHoursUsed": 249191188022.46954,
        "byteUsageMax": null,
        "breakdown": {
          "idle": 0.9689472045060643,
          "other": 0,
          "system": 0.031052795493935748,
          "user": 0
        },
        "adjustment": 0,
        "totalCost": 1.190033,
        "storageClass": "",
        "volumeName": "",
        "claimName": "",
        "claimNamespace": ""
      },
      "Node": {
        "type": "Node",
        "properties": {
          "category": "Compute",
          "provider": "AWS",
          "service": "Kubernetes"
        },
        "labels": {
          "label_beta_kubernetes_io_os": "linux",
          "label_eks_amazonaws_com_capacityType": "ON_DEMAND",
          "label_eks_amazonaws_com_sourceLaunchTemplateVersion": "1",
          "label_kubernetes_io_os": "linux"
        },
        "window": {
          "start": "2022-11-07T00:00:00Z",
          "end": "2022-11-08T00:00:00Z"
        },
        "start": "2022-11-07T00:00:00Z",
        "end": "2022-11-07T20:22:00Z",
        "minutes": 1222,
        "nodeType": "",
        "cpuCores": 7.996727,
        "ramBytes": 24359995365.18494,
        "cpuCoreHours": 162.866667,
        "ramByteHours": 496131905604.26666,
        "GPUHours": 0,
        "cpuBreakdown": {
          "idle": 0.8225574364538241,
          "other": 0.016029761260400448,
          "system": 0.019269797625607796,
          "user": 0.14214300466016733
        },
        "ramBreakdown": {
          "idle": 0.8991634628362297,
          "other": 0,
          "system": 0.008285625609704916,
          "user": 0.09255091155406577
        },
        "preemptible": 0,
        "discount": 0,
        "cpuCost": 3.88239,
        "gpuCost": 0,
        "gpuCount": 0,
        "ramCost": 1.606193,
        "adjustment": 0,
        "totalCost": 5.488583
      }
    }
  ]
}
```
