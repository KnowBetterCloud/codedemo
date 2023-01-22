# Create a VPC with 3 x AZ and  Private Subnets

https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html

```
MY_EKS_CLUSTER="$PROJECT"
MY_REGION="us-east-1"
MY_VERSION="1.24"
STACK_NAME="${MY_EKS_CLUSTER}"
```

NOTE:  I am directing you to pull a file from *my* Git Repo... Therefore, I urge caution, and then advise you to compare my file to the AWS example.   
Changes I made:
* Added *-03 subnets (pub and priv)
* Modified the CIDR from a /18 to a /19 (16,384 to 8,192) for each subnet

```
curl -o amazon-eks-vpc-private-subnets.yaml https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml
curl -o amazon-eks-vpc-3-private-subnets.yaml https://raw.githubusercontent.com/cloudxabide/Mr_MeeseEKS/main/Files/amazon-eks-vpc-3-private-subnets.yaml
sdiff amazon-eks-vpc-private-subnets.yaml amazon-eks-vpc-3-private-subnets.yaml
```

```
aws cloudformation create-stack --stack-name "${STACK_NAME}" \
  --template-body file://amazon-eks-vpc-3-private-subnets.yaml \
  --region ${MY_REGION}
```

Check the status of the Cloudformation Stacks 
NOTE: there will be many as all the stacks start with "$PROJECT" (codedemo)
```
aws cloudformation list-stacks --query "StackSummaries[?starts_with(StackName, '${PROJECT}')].{StackName:StackName,StackStatus:StackStatus} | sort_by(@, &StackName)"
```
