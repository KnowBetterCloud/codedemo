# Create EKS Cluster


### OK.. now on to the cluster
```
# Get the VPC_ID for the mrmeseeks VPC we created
MY_VPC_ID=$(aws ec2 describe-vpcs --filters "Name=tag:Name,Values=codedemo-VPC" --query "Vpcs[].VpcId" --output=text)

# Gather the SubnetId for the private subnets
SUBNETS=$(aws ec2 describe-subnets --region $MY_REGION  --filters "Name=vpc-id,Values=${MY_VPC_ID}" --query 'Subnets[?MapPublicIpOnLaunch==`false`].SubnetId' --output=text)
```

The following command is not async - i.e. it will keep providing output until it is complete.  Enjoy!  
It will likely take 15~25 minutes
```
eksctl create cluster --name ${MY_EKS_CLUSTER} --region ${MY_REGION} --version ${MY_VERSION} --vpc-private-subnets $(echo $SUBNETS | sed 's/ /,/g') --without-nodegroup
```


```
aws cloudformation list-stacks --query 'StackSummaries[?starts_with(StackName, `eksctl-codedemo-cluster`)].{StackName:StackName,StackStatus:StackStatus} | sort_by(@, &StackName)'
```

