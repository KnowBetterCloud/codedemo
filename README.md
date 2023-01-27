# codedemo
Deploy EKS Cluster and sample apps.

At the highest-level, this repo will guide you through:

* Create EKS cluster (using EC2 instances and Managed Node Groups)
* Deploy a Demo App (crystal, node-js) that provides a visual outlook of how pods run on several nodes (and in several Availability Zones)
* Deploy a Java App by building a Software Pipeline using AWS Developer Tools

## Notes, Caveats, Warnings
This "project" is to deploy a single EKS cluster and a few apps.  If you are using an existing AWS account, you may have a different experience.  
I tried to make the code retrieve data specific to the project we are working with (codedemo, in this case), but it may not "filter" adequetely 
if there happens to be lots of other resources in your account.

This code was not created to be:

* extremely efficient
* idempotent
* used in/for Production workloads

it is *intentionally* inefficient, overly-verbose, and essentially non-scriptable for a reason - so that you have to cut-and-paste the code, see what it is doing, etc...

## Demo Information
Here is a list of "variables" and their values that I will use throughout this particular demo.

I will create/maintain a file to "source" with all the variables I need.

| Variab le         | Purpose                                | Value                   | 
|:------------------|:---------------------------------------|:------|
| MY_APP_NAME       | Represent the app being deployed       | ecsdemo-frontend <BR> aws-proserve-java-greeting-dev |
| MY_ROJECT         | Name of this "project"                 | codedemo |
| MY_PARENT_DOMAIN  | Top Level Domain (TLD)                 | clouditoutloud.com |
| MY_PROJECT_DOMAIN | Domain specifically for *this* project | $MY_PROJECT.$MY_PARENT_DOMAIN <br> ex. codedemo.clouditoutloud.com |

NOTE:  As annoying as seeing a bunch of variables starting with MY_ is, I *do* have a reason.  I want source the variables file, if it exists, and then be able to run 
```
set | grep MY_
# or
env | grep MY_
```

## Create Cloud9 Environment
[AWS Cloud9](https://aws.amazon.com/cloud9/) is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. 
Some cool advantages:
* Cloud9 integrates with AWS Services
* You don't need to manage AWS Access Keys
* You only pay for what you use. (Cloud9 will "pause" after use, and then spin back up when you need it later)

[Create your Cloud9 Environment](Create_Cloud9_Environment.md)  

Add verbiage to Source Environment Variables for this Code Demo for your login shell
``` 
curl -o variables.txt https://raw.githubusercontent.com/KnowBetterCloud/codedemo/main/Files/variables.txt
echo "
# Sourcing Project Vars
[ -f ${HOME}/environment/variables.txt ] && . ${HOME}/environment/variables.txt"  | tee -a ${HOME}/.bashrc
. ${HOME}/.bashrc
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

## Deploy Kubernetes Dashboard
Follow this guide to [Deploy Official Kubernetes Dashboard](./Deploy_Kubernetes_Dashboard.md)

## Deploy Visualization Demo App
This app is really cool.  It provides a "visual representation" of where the 3-tiers of pods are running.

[Deploy Demo Visualization App - 3-tier](./Deploy_Demo_Visualization_App.md)

## Deploy Software Pipeline Demo App
[Deploy Software Pipeline Demo](./Deploy_Software_Pipeline.md)

## Extra Credit - CNAMEs for your Apps
[Create CNAME for Apps](./Create_CNAMES.md)

## Upgrade Cluster Version
Upgrade your cluster from 1.23 to 1.24 (as of 2023-01-24)
[Upgrade your EKS Cluster](./Upgrade_EKS.md)


## Undo this
[Uninstall the Codedemo Resources](./Uninstall.md)

## TODO
I have a few things [TODO](./TODO.md)

