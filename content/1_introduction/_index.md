---
title : "Introduction"
chapter: true
weight : 10
---
# What is Kubecost?

[Kubecost](https://www.kubecost.com/) provides real-time cost visibility and insights for teams using Kubernetes.

Installing Kubecost on a single cluster or a fleet of clusters uncovers patterns that create overspending on infrastructure and helps teams prioritize where to focus optimization efforts. By identifying root causes for negative patterns, customers using Kubecost save up to 80% of their Kubernetes cloud infrastructure costs. Today, Kubecost empowers thousands of teams across companies of all sizes to monitor and reduce costs, while balancing cost, performance, and reliability.

Kubecost is tightly integrated with the open source Cloud Native ecosystem and built for engineers and developers first, making it easy to drive adoption within any organization.

# What is OpenCost?

OpenCost is a vendor-neutral open source project for measuring Kubernetes costs in near real time. The project is a combination of the OpenCost core allocation engine with a new community-driven spec on how to monitor costs in Kubernetes.
Learn more about [OpenCost](https://www.opencost.io/).

OpenCost UI:
![opencost-ui](/images/opencost-ui.png)

OpenCost API:

![opencost-api](/images/opencost-api.png)

# How is OpenCost different from Kubecost?

|  | **OpenCost** | **Kubecost Free** |
|---|:---:|:---:|
| Description | An open source tool for monitoring cost, resource allocation, and utilization data in Kubernetes | Feature-rich, always-free, Kubecost uses an Open Core model that builds on OpenCost with greater scaling and cost savings opportunities |
| Best for | Small clusters, simple API-based reporting needs | Larger clusters / Detailed UI |
| Price | Free | Free |
| Clusters supported | Unlimited (best for smaller node-counts) | Single cluster-at-a-time view |
| Number of nodes | Unlimited | Unlimited |
| Deployment | Deployed as a pod. Prometheus and kube-state-metrics dependencies  need to be managed separately. | Deployed using Helm. Turn-key solution with Prometheus and Grafana. |
| Metric retention | Limited by Prometheus environment | 15 days |
| Support | Built and supported by community users. Interact with our vibrant community to quickly and easily learn, build, and deploy OpenCost for Kubernetes cost monitoring! | Ticket-based support dedicated to resolving issues quickly and efficiently. Available 9:00 AM - 8:00 PM EST |

Learn more about the different Kubecost versions and get started [here](https://www.kubecost.com/pricing/).

**Adobe User Story** - What is the value of OpenCost vs Kubecost?

# Kubecost architecture diagram

![kubecost-arch](/images/kubecost-arch.png)
