

* Routing rule
host: app1.lingaiah.online
path: /
service: app1:80


* So traffic flow is:

Browser
  → ALB
    → Target Group
      → Service app1
        → Pods (port 80)


* As-is, this works perfectly for:

http://app1.lingaiah.online



# Important Note: Please add this policy ‘ElasticLoadBalancingFullAccess’ to EKS Worker node's IAM role and execute it.

# Ingress Controller - AWS Load Balancer Controller installation
https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.8/deploy/installation/

1. Create an IAM OIDC provider. You can skip this step if you already have one for your cluster.
```
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster expense \
    --approve
```

2. Download an IAM policy for the LBC using one of the following commands:
If your cluster is in any other region:
```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.10.0/docs/install/iam_policy.json
```

3. Create an IAM policy named AWSLoadBalancerControllerIAMPolicy. If you downloaded a different policy, replace iam-policy with the name of the policy that you downloaded.
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

4. Create an IAM role and Kubernetes ServiceAccount for the LBC. Use the ARN from the previous step.
```
eksctl create iamserviceaccount \
--cluster=expense \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::484907532817:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve
```

```
aws eks update-kubeconfig --region us-east-1 --name expense
```

Helm Installation Steps:
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```
```
chmod 700 get_helm.sh
```
```
./get_helm.sh
```

5. Add the EKS chart repo to Helm
```
helm repo add eks https://aws.github.io/eks-charts
```

6. Helm install command for clusters with IRSA:
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=expense --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```

* check aws-load-balancer-controller is running in kube-system namespace.
```
kubectl get pods -n kube-system
```