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
## add to file 
```
    spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=basic-cluster
        - --aws-vpc-id=vpc-03468a8157edca5bd
        - --aws-region=us-east-1
```
## Deploy  2048 game
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-deployment.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-service.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-ingress.yaml
```
## Get host 
```
kubectl get ingress/2048-ingress -n 2048-game
```
## create secret
### Base 64 encoded
```
echo -n 'my-app' | base64
echo -n '39528$vdg7Jb' | base64
```
### file
```
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
data:
  username: bXktYXBw
  password: Mzk1MjgkdmRnN0p
```
### with out base64
```
kubectl create secret generic test-secret --from-literal=username='my-app' --from-literal=password='39528$vdg7Jb'
```
### For horizontal scaling 
```
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```
