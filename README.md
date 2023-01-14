# Cost Optimization with Kubecost

In this workshop, you will learn how use Kubecost Cost Analyzer tool to measure the cost allocation of various resources in your EKS cluster. You will also learn how to access the same information using the Kubecost APIs and integrate Kubecost with Amazon Managed Prometheus (AMP) for monitoring at scale.

## Building the Website

This page is built with Hugo, so you'll need it [installed](https://gohugo.io/getting-started/quick-start/#step-1-install-hugo)

First, clone this repo:

```bash
git clone git@github.com:aws-samples/aws-modernization-with-kubecost.git
```
Ensure you've also cloned the submodules:

```bash
git submodule init
git submodule update
```

Then serve the website with hugo:

```bash
hugo server

```

### Learning Objectives
- What is Kubecost and OpenCost?
- How to deploy it on EKS Cluster
- How to use Kubecost to monitor and optimize cluster spending