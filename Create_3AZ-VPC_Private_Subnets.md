# Create a VPC with 3 x AZ and  Private Subnets

https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html

```
MY_EKS_CLUSTER="$APP_NAME"
MY_REGION="us-east-1"
MY_VERSION="1.24"
STACK_NAME="${MY_EKS_CLUSTER}"
```

NOTE:  I am directing you to pull a file from *my* Git Repo... So, I urge caution, and then advise you to compare to the AWS example. 
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

# So.... I can't figure out how to pass a $VARIABLE in to the following command
```
aws cloudformation list-stacks --query 'StackSummaries[?starts_with(StackName, `codedemo`)].{StackName:StackName,StackStatus:StackStatus} | sort_by(@, &StackName)'

```
