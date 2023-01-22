# Create a Managed NodeGroup 

## You don't *need* to do this.  Unless you're *real* curious, like me.  
```
MY_REGION=us-east-1
aws ec2 create-key-pair --region $MY_REGION --key-name $APP_NAME --query 'KeyMaterial' --output text > $APP_NAME.pem
aws ec2 describe-key-pairs --key-name $APP_NAME 
```

Get the EKS cluster name if not already set
```
[ -z $MY_EKS_CLUSTER ]  &&  { MY_EKS_CLUSTER=$(aws eks list-clusters --query 'clusters' --output text); }
```

## Amazon Linux 2 (AL2) AMI
```

eksctl create nodegroup \
  --cluster $MY_EKS_CLUSTER \
  --region $MY_REGION \
  --name my-mng-al2 \
  --node-ami-family AmazonLinux2 \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 4 \
  --ssh-access \
  --ssh-public-key $APP_NAME \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --managed \
  --node-private-networking
```

## Bottlerocket AMI
```
eksctl create nodegroup \
  --cluster $MY_EKS_CLUSTER \
  --region $MY_REGION \
  --name my-mng-bottlerocket \
  --node-ami-family Bottlerocket \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 4 \
  --ssh-access \
  --ssh-public-key $APP_NAME \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --managed \
  --node-private-networking
```

## Some status reporting
```
kubectl get nodes -o wide
aws eks list-nodegroups --cluster-name $MY_EKS_CLUSTER
eksctl get nodegroup --cluster $MY_EKS_CLUSTER
```
