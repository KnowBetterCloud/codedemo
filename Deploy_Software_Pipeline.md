# Deploy Software Pipeline
Status:  This just "notes" at this point (Jan 2023)
          I will improve the formatting so this is easier to follow

I am using the following naming "standard"  
"${MY_PROJECT}-${MY_DATE}-${MY_VERSION}"

ex codedemo-20230106-01

CodeCommit Repo
CodePipeline 
S3 Bucket

## Variables for this Demo
```
[ -z $MY_PROJECT ] && { echo "Sourcing Project Variables"; . ./variables.txt; }
echo "Using Project: ${MY_PROJECT}"
echo "Deployed in Region: $MY_REGION"
```

## Create a Project Directory and manage code there.  Create S3 bucket to copy code to.
```
mkdir -p ${HOME}/Projects/${MY_PROJECT}/; cd $_
wget https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/samples/p-attach/95a5b5c2-d7fb-41eb-9089-455318c0d585/attachments/attachment.zip
unzip attachment.zip 
unzip java-cicd-eks-cfn.zip   
```

### Update aws-proserve-java-greeting code - replace the "default" repo URL with our repo URL
NOTE:  This expect GNU-sed (MacOS sed will not work)
-- confirm this works, else: THIS NEEDS TO BE DONE VIA CONSOLE YET - I.e I need to rework this code to add to app_code.
```
[ -e $MY_ACCOUNT_ID ] && { export MY_ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output=text); }
cd version1/app_code
unzip -o app_code.zip 
sed -i -e "s/509501787486/${MY_ACCOUNT_ID}/g" $(find . -name values.dev.yaml)
grep dkr $(find . -name values.dev.yaml)
sed -i -e "s/us-east-2/${MY_REGION}/g" $(find . -name values.dev.yaml)
grep dkr $(find . -name values.dev.yaml)
zip -r app_code.zip app aws-proserve-java-greeting/ buildspec_deploy buildspec.yml
cd -
```

Create S3 Buck and upload updated code
```
aws s3 mb s3://${MY_S3_BUCKET_NAME} --region $MY_REGION
# NOTE: we need to create a folder ("Code") which will happen when we cp bits up to the bucket
aws s3 cp $(find . -name app_code.zip) s3://${MY_S3_BUCKET_NAME}/Code/   --region $MY_REGION
aws s3 ls s3://${MY_S3_BUCKET_NAME}/Code/ --region $MY_REGION
```

## Create the Stack which will create 
  * ECR repo
  * CodeCommit repo
```
STACK_NAME="${MY_PROJECT}-codecommit-ecr"
STACK_PARAMETERS=${STACK_NAME}.cfn
STACK_PARAMETERS_TMP=${STACK_NAME}.cfn.tmp

# Create Human-Readable "params", then parse it in to json 
cat << EOF > ${STACK_PARAMETERS_TMP}
CodeCommitRepositoryName              ${MY_PROJECT}
ECRRepositoryName	              ${MY_PROJECT}
CodeCommitRepositoryS3BucketObjKey    Code/app_code.zip
CodeCommitRepositoryBranchName	      master	
CodeCommitRepositoryS3Bucket	      ${MY_S3_BUCKET_NAME}
EOF

grep -v ^# $STACK_PARAMETERS_TMP | while read ParameterKey ParameterValue DUMMY; do echo -e "  {\n    \"ParameterKey\": \"${ParameterKey}\",\n    \"ParameterValue\": \"${ParameterValue}\"\n  },"; done > ${STACK_PARAMETERS}
#- once you have the params.json created, add open/close square-brackets at the beginning and end
case `sed --version | head -1` in
  *GNU*)
    # Remove the entries which are just a hypen "-"
    sed -i -e 's/"-"/""/g' ${STACK_PARAMETERS}
    # Add the '[' to the beginning of the file
    sed -i -e '1i[' ${STACK_PARAMETERS}
    # Remove the trailing ',' from the last entry
    sed  -i '$ s/},/}/' ${STACK_PARAMETERS}
  ;;
  *)
sed -i'' '1i\
[\
' ${STACK_PARAMETERS}
  ;;
esac
echo "]" >> ${STACK_PARAMETERS}
```

# Validate the provided template 
```
aws cloudformation validate-template --template-body file://$(find . -name codecommit.yaml) --region $MY_REGION
aws cloudformation create-stack --stack-name "${STACK_NAME}" \
  --template-body file://$(find . -name codecommit.yaml) \
  --parameters file://${STACK_PARAMETERS} \
  --region $MY_REGION
```

## Gather ECR Information
```
ECR_URL=$(aws ecr describe-repositories  --query "repositories[?contains(repositoryUri,'${MY_PROJECT}')].{repositoryUri:repositoryUri}" --output text --no-cli-pager )
ECR_URL_BASE=$(echo $ECR_URL  | grep codedemo | cut -f1 -d\/  )
echo "ECR_URL: $ECR_URL"
echo "ECR_URL_BASE: $ECR_URL_BASE"

aws codecommit list-repositories --region $MY_REGION --query "repositories[].repositoryName" --output text
```

CFN Template to deploy CodePipeline to build Docker Image of java application and push to ECR and deploy to EKS
```
STACK_NAME="${MY_PROJECT}-codepipeline"
STACK_PARAMETERS=${STACK_NAME}.cfn
STACK_PARAMETERS_TMP=${STACK_NAME}.cfn.tmp
```

TODO: Need to get this to query for the cluster name 
- MY_EKS_CLUSTER_NAME=$(aws eks list-clusters --query "clusters" --output text)
```
MY_EKS_CLUSTER_NAME=codedemo
cat << EOF > ${STACK_PARAMETERS_TMP}
EKSCodeBuildAppName	aws-proserve-java-greeting
EcrDockerRepository	${MY_PROJECT}
SourceRepoName ${MY_PROJECT}
CodeBranchName	master
EKSClusterName	$MY_EKS_CLUSTER_NAME
EnvType dev 
EOF
```

```
grep -v ^# $STACK_PARAMETERS_TMP | while read ParameterKey ParameterValue DUMMY; do echo -e "  {\n    \"ParameterKey\": \"${ParameterKey}\",\n    \"ParameterValue\": \"${ParameterValue}\"\n  },"; done > ${STACK_PARAMETERS}
#- once you have the params.json created, add open/close square-brackets at the beginning and end
case `sed --version | head -1` in
  *GNU*)
    # Remove the entries which are just a hypen "-"
    sed -i -e 's/"-"/""/g' ${STACK_PARAMETERS}
    # Add the '[' to the beginning of the file
    sed -i -e '1i[' ${STACK_PARAMETERS}
    # Remove the trailing ',' from the last entry
    sed  -i '$ s/},/}/' ${STACK_PARAMETERS}
  ;;
  *)
sed -i'' '1i\
[\
' ${STACK_PARAMETERS}
  ;;
esac
echo "]" >> ${STACK_PARAMETERS}
```

# Validate the provided template, then create stack using it
```
aws cloudformation validate-template --template-body file://$(find . -name build_deployment.yaml)
aws cloudformation create-stack --stack-name "${STACK_NAME}" \
  --template-body file://$(find . -name build_deployment.yaml) \
  --parameters file://${STACK_PARAMETERS} \
  --capabilities CAPABILITY_NAMED_IAM \ 
  --region $MY_REGION
```

# Confirm kubeconfig is able to query for cluster
```
(eksctl get clusters) || { aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region ${MY_REGION}; }
aws cloudformation --region=${MY_REGION} describe-stacks --query "Stacks[?StackName=='${MY_PROJECT}-codepipeline'].Outputs[0].OutputValue" --output text

## You need to wait until the previous command provides output
bash $(find . -name kube_aws_auth_configmap_patch.sh) $(aws cloudformation --region=${MY_REGION} describe-stacks --query "Stacks[?StackName=='${MY_PROJECT}-codepipeline'].Outputs[0].OutputValue" --output text)
```

## Update Code 
We are using code created back in 2019/2020.  And Trivy, for example, has changed versions and allowed command line parameters.

In CodeCommit Console, check out CodeCommit | Repositories | Code  
codedemo/buildspec.yml  
change/update  
      - trivy --no-progress --exit-code 1 --severity HIGH,CRITICAL --auto-refresh $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$CODEBUILD_RESOLVED_SOURCE_VERSION
to   
      - trivy image $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$CODEBUILD_RESOLVED_SOURCE_VERSION

OR...
      - trivy --timeout 5m image $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$CODEBUILD_RESOLVED_SOURCE_VERSION

Then... approve (in CodePipeline)  

## Check out the end of the CodeBuild and revie teh logs.  At teh very end is the endpoint you can acces (or review EC2: LoadBalancers)  

Also (I have no clue how the versioning works, or why it matters)  
codedemo/app/pom.xml  
project: parent: version: 2.0.5.RELEASE  
to  
project: parent: version: 2.0.9.RELEASE  
or...  
project: parent: version: 2.7.7.RELEASE  


# Push AmazonCorretto image to your repo (This is still a WIP)
```
ECR_URL=$(aws ecr describe-repositories  --query "repositories[?contains(repositoryUri,'${MY_PROJECT}')].{repositoryUri:repositoryUri}" --output text --no-cli-pager )
ECR_URL_BASE=$(echo $ECR_URL  | grep codedemo | cut -f1 -d\/  )
ECR_REPOSITORY_NAME=$(aws ecr describe-repositories --query "repositories[].repositoryName" --output text)
echo "ECR_URL= $ECR_URL"
echo "ECR_URL_BASE= $ECR_URL_BASE"
aws ecr get-login-password --region $MY_REGION | docker login --username AWS --password-stdin $ECR_URL_BASE

# registry/repository:tag
IMAGE=amazoncorretto:8-alpine
REPOSITORY=$(echo $IMAGE | cut -f1 -d\:)
IMAGE_TAG=$(echo $IMAGE | cut -f2 -d\:)
docker pull $IMAGE

IMAGE_ID=$(docker images | grep $IMAGE_TAG | awk '{ print $3 }')
echo "docker tag $IMAGE_ID $ECR_URL:$IMAGE_TAG"
docker tag $IMAGE $ECR_URL:$IMAGE_TAG
echo "docker push $ECR_URL:$IMAGE_TAG"
docker push $ECR_URL:$IMAGE_TAG

# The following... works (just an example)
{ docker tag amazoncorretto:8-alpine $ECR_URL:latest
docker push $ECR_URL:latest }
```

Change Directoy back to ~/environment
```
cd ~/environment
```

### RANDOM BITS and COMMANDS
```
aws eks list-clusters --query "clusters[]" --output text --no-cli-pager
aws codecommit list-repositories --query "repositories[].repositoryName" --output=text
aws cloudformation --region=us-east-1 describe-stacks --query "Stacks[?StackName=='codedemo-20231015-pipeline'].Outputs[0].OutputValue" --output text
```

#### Customize your code (simialr to updating the account # - which *should* be working now - above)
So, there is an version1/app_code dir that contains an app_code.zip
You would want to:   
* make a backup of app (mv app app.bak   
unzip app_code.zip   
vi app/src/main/java/org/aws/samples/greeting/GreetingController.jav  
update the line with "return"  
zip app_code.zip app  
THEN.... copy it to s3  

The trigger is implemented via: "PollForSourceChanges - App"
