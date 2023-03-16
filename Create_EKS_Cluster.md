# Create EKS Cluster

NOTE:  You have to ensure that your VPC has been created (check Cloudformation)

### OK.. now on to the cluster
```
# Get the VPC_ID for the $MY_PROJECT VPC we created
MY_VPC_ID=$(aws ec2 describe-vpcs --region $MY_REGION --filters "Name=tag:Name,Values=${MY_PROJECT}-VPC" --query "Vpcs[].VpcId" --output=text)

# Gather the SubnetId for the private subnets
SUBNETS=$(aws ec2 describe-subnets --region $MY_REGION  --filters "Name=vpc-id,Values=${MY_VPC_ID}" --query 'Subnets[?MapPublicIpOnLaunch==`false`].SubnetId' --output=text)

echo "$MY_VPC_ID"
echo "$SUBNETS"

```


The following command is not async - i.e. it does not return the command prompt and it will keep providing output until it is complete.  Enjoy!  
It will likely take 15~25 minutes - and you can watch the progress in the Cloudformation | Stacks Console 
```
eksctl create cluster --name ${MY_EKS_CLUSTER_NAME} --region ${MY_REGION} --version ${MY_EKS_VERSION} --vpc-private-subnets $(echo $SUBNETS | sed 's/ /,/g') --without-nodegroup
```

Get the Status of the Cloudformation Stack
```
aws cloudformation list-stacks --region $MY_REGION --query "StackSummaries[?starts_with(StackName, 'eksctl-${MY_PROJECT}-cluster')].{StackName:StackName,StackStatus:StackStatus} | sort_by(@, &StackName)"
```

## Troubleshooting
Note:  you will receive a "warning" about having only 1 stack. That is expected at this point. You will create the managed node group in a future step.
```
eksctl utils describe-stacks --region=us-east-1 --cluster=codedemo
```
