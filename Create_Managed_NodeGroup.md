# Create a Managed NodeGroup 

## You don't *need* to do this.  Unless you're *real* curious, like me.  
```
[ -e $MY_REGION ] && MY_REGION=us-east-1
aws ec2 create-key-pair --region $MY_REGION --key-name $MY_PROJECT --query 'KeyMaterial' --output text > $MY_PROJECT.pem
aws ec2 describe-key-pairs --region $MY_REGION --key-name $MY_PROJECT 
```

Get the EKS cluster name if not already set
```
[ -z $MY_EKS_CLUSTER_NAME ]  &&  { MY_EKS_CLUSTER_NAME=$(aws eks list-clusters --query 'clusters' --output text); }
echo "EKS Cluster: $MY_EKS_CLUSTER_NAME"
```

NOTE: this activity will create a Cloudformation Stack for each Managed Node Group  
Stack name: eksctl-codedemo-nodegroup-my-mng-al2 (example)

## Amazon Linux 2 (AL2) AMI
```

eksctl create nodegroup \
  --cluster $MY_EKS_CLUSTER_NAME \
  --region $MY_REGION \
  --name my-mng-al2 \
  --node-ami-family AmazonLinux2 \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 4 \
  --ssh-access \
  --ssh-public-key $MY_PROJECT \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --managed \
  --node-private-networking

kubectl get nodes -o wide
```

## Bottlerocket AMI
```
eksctl create nodegroup \
  --cluster $MY_EKS_CLUSTER_NAME \
  --region $MY_REGION \
  --name my-mng-bottlerocket \
  --node-ami-family Bottlerocket \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 4 \
  --ssh-access \
  --ssh-public-key $MY_PROJECT \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --managed \
  --node-private-networking

kubectl get nodes -o wide
```

## Some status reporting
```
kubectl get nodes -o wide
aws eks list-nodegroups --region $MY_REGION --cluster-name $MY_EKS_CLUSTER_NAME
eksctl get nodegroup --region $MY_REGION --cluster $MY_EKS_CLUSTER_NAME
```
