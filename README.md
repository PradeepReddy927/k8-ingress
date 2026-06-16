1.  Install OIDC Provider
``` 
eksctl utils associate-iam-oidc-provider \
  --region us-east-1 \
  --cluster roboshop-dev \
  --approve
```  

2. Download IAM Policy
```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.16.0/docs/install/iam_policy.json
``` 

3. Create IAM policy
```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
``` 

4. Create Service account and IAM role. map the above policy
```
eksctl create iamserviceaccount \
--cluster=roboshop-dev \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::267834697821:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve
```

### Install Drivers
Install Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
```

Add EKS chart repo
```
helm repo add eks https://aws.github.io/eks-charts
```
Install Load balancer controller
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=roboshop-dev --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```
check drivers are running
```
kubectl get pods -n kube-system
```

----------

1. Disable Termination Protection
```
aws cloudformation update-termination-protection \
  --no-enable-termination-protection \
  --stack-name eksctl-roboshop-dev-addon-iamserviceaccount-kube-system-aws-load-balancer-controller \
  --region us-east-1
```
2. Delete the Stack
```
aws cloudformation delete-stack \
  --stack-name eksctl-roboshop-dev-addon-iamserviceaccount-kube-system-aws-load-balancer-controller \
  --region us-east-1
```
3. Wait for Deletion
```
aws cloudformation wait stack-delete-complete \
  --stack-name eks
```
------

1. Delete IAM Policy
First get the policy ARN
```
aws iam list-policies --scope Local
```
Then delete the policy
```
aws iam delete-policy \
  --policy-arn arn:aws:iam::267834697821:policy/AWSLoadBalancerControllerIAMPolicy
```
2. Delete OIDC Provider
```
To find the OIDC provider
```
aws iam list-open-id-connect-providers
```
Get details
```
aws iam get-open-id-connect-provider \
  --open-id-connect-provider-arn <OIDC_PROVIDER_ARN>
```
Delete it
```
aws iam delete-open-id-connect-provider \
  --open-id-connect-provider-arn <OIDC_PROVIDER_ARN>

aws iam delete-open-id-connect-provider \
  --open-id-connect-provider-arn arn:aws:iam::267834697821:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/790C3712534B7A926124C8CA2CF4A389  
```
Verify deletion
```
aws iam list-open-id-connect-providers
```
