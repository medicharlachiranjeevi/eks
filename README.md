# cluster eksctl put it in cluster.yaml

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: basic-cluster
  region: us-east-1

nodeGroups:
  - name: ng-1
    instanceType: t2.medium
    minSize: 1
    maxSize: 4
    desiredCapacity: 1
    ssh:
      allow: true # will use ~/.ssh/id_rsa.pub as the default ssh key
```
## To create cluster 
```
eksctl create cluster --config-file=cluster.yaml

```
## create policy ALBIngressControllerIAMPolicy
```

aws iam create-policy \
    --policy-name ALBIngressControllerIAMPolicy \
    --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json
```

## associate iam for eks

```
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster basic-cluster \
    --approve
```

## Attach policy to nodes to create Elb
```
eksctl create iamserviceaccount \
    --region us-east-1 \
    --name alb-ingress-controller \
    --namespace kube-system \
    --cluster basic-cluster \
    --attach-policy-arn arn:aws:iam::********:policy/ALBIngressControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
 ```
## Install aws-alb-ingress-controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml
```
## Deploy ALB Ingress Controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/alb-ingress-controller.yaml
```
## Edit deploy file
```
kubectl edit deployment.apps/alb-ingress-controller -n kube-system
```
## add 
```
    spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=prod
        - --aws-vpc-id=vpc-03468a8157edca5bd
        - --aws-region=us-east-1
```
