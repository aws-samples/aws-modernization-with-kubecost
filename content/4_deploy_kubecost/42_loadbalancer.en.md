---
title : "Expose Kubecost with AWS Load Balancer Controller"
chapter: true
weight : 42
---

### Overview

In this module you can learn how to expose Kubecost for external access using AWS Load Balancer Controller (ALB). The AWS Load Balancer Controller manages AWS Elastic Load Balancers for a Kubernetes cluster. The controller provisions the following resources:

- An AWS Application Load Balancer (ALB) when you create a Kubernetes Ingress.

- An AWS Network Load Balancer (NLB) when you create a Kubernetes service of type LoadBalancer.

In this module, the Kubecost is exposed via a Kubernetes ingress and an AWS Application Load Balancer.

### Reference Architecture diagram:

![kubecost-aws-alb](/images/kubecost-aws-alb.png)

### Instructions

On the same Amazon EKS cluster that you installed Kubecost, run the following command (copy & paste the entire code block) to create an Ingress object:

```bash
cat <<'EOF' |
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubecost-alb-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  rules:
    - http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: kubecost-cost-analyzer
              port:
                number: 9090
EOF
(export NAMESPACE=kubecost && kubectl apply -n $NAMESPACE -f -)
```

After few seconds, an AWS application load balancer is created. You can run the following command to get the DNS name for accessing Kubecost user interface (UI):

```bash
export ENDPOINT=$(kubectl get ingress kubecost-alb-ingress -n kubecost --output jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "Kubecost UI DNS name: ${ENDPOINT}"
```
Example result:

```bash
Kubecost UI DNS name: k8s-kubecost-kubecost-xxxxxxxxxx-xxxxxxxxxx.us-west-2.elb.amazonaws.com
```

On your web browser, navigate to ${ENDPOINT} to access the dashboard. 

> **In some cases, you may need to wait for few minutes before accessing the Kubecost UI to allow AWS completely provisioning the application load balancer properly**

You can learn more about Kubecost features in the next module.
