# codedemo
Deploy EKS Cluster and sample apps.

At the highest-level, this repo will guide you through:

* Create EKS cluster (using EC2 instances and Managed Node Groups)
* Deploy a Demo App (crystal, node-js) that provides a visual outlook of how pods run on several nodes (and in several Availability Zones)
* Deploy a Java App by building a Software Pipeline using AWS Developer Tools

## Demo Information
Here is a list of "variables" and their values that I will use throughout this particular demo.

I will create/maintain a file to "source" with all the variables I need.

| Variable      | Value                 |
|:--------------|:----------------------|
| APP_NAME      | codedemo |
| PROJECT       | \$APP_NAME+\`date +%F\` <br> ex. codedemo-2023-01-20 |
| PARENT_DOMAIN | clouditoutloud.com |
| APP_DOMAIN    | $APP_NAME.$PARENT_DOMAIN <br> ex. codedemo.clouditoutloud.com |

## Create Cloud9 Environment
[AWS Cloud9](https://aws.amazon.com/cloud9/) is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. 
Some cool advantages:
* Cloud9 integrates with AWS Services
* You don't need to manage AWS Access Keys
* You only pay for what you use. (Cloud9 will shutdown after use, and then spin back up when you need it later)

[Create your Cloud9 Environment](Create_Cloud9_Environment.md)  

Source Environment Variables for this Code Demo
``` 
curl -o variables.txt https://raw.githubusercontent.com/KnowBetterCloud/codedemo/main/Files/variables.txt
. ./variables.txt
```
Run through the [Install Tools](Install_Tools.md) doc  
[Modify IAM settings for your Workspace](./Modify_IAM_Settings.md)

## Create EKS Cluster

### But first, create a VPC
[Create a VPC with 3 x AZ and Private Subnets](./Create_3AZ-VPC_Private_Subnets.md)

### Create the Cluster

[Create an EKS Cluster](./Create_EKS_Cluster.md)

### Create a Managed Node Group
The following contains the procedure to create either an Amazon Linux 2 (AL2) or Bottlerocket Managed Node Group

[Create a Managed Node Group](./Create_Managed_NodeGroup.md)

## Deploy Visualization Demo App

## Deploy Pipeline Demo App



