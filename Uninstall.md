# Uninstall

This is essentially some of the "manual" steps you likely have to do before delete the Cloudformation Stacks.

# Running Container Apps
## Undeploy Visualization App
```
cd ~/environment/ecsdemo-frontend
kubectl delete -f kubernetes/service.yaml
kubectl delete -f kubernetes/deployment.yaml

cd ~/environment/ecsdemo-crystal
kubectl delete -f kubernetes/service.yaml
kubectl delete -f kubernetes/deployment.yaml

cd ~/environment/ecsdemo-nodejs
kubectl delete -f kubernetes/service.yaml
kubectl delete -f kubernetes/deployment.yaml
```

## Delete route53 CNAMES
*TODO*
export HOSTED_ZONE_ID=$(aws route53 list-hosted-zones-by-name |  jq --arg name "$MY_PROJECT_DOMAIN." -r '.HostedZones | .[] | select(.Name=="\($name)") | .Id' )
curl -o cname_template_delete.json https://raw.githubusercontent.com/KnowBetterCloud/codedemo/main/Files/cname_template_delete.json

for MY_APP_NAME in aws-proserve-java-greeting-dev ecsdemo-frontend
do 
  envsubst < cname_template.json > cname_${MY_APP_NAME}_delete.json
  aws route53 change-resource-record-sets --hosted-zone-id $HOSTED_ZONE_ID --change-batch file://cname_${MY_APP_NAME}_delete.json
done
  
## Delete IAM role(s)
*TODO*
NOTE:  This *may* need to be done via Console?  I.e. if I do this *before* I delete the Cloud9 instance, will I be removing the capability to do the rest of the deletes?  (I think Yes.. and therefore I should move this to the bottom)

## Delete NodeGroup
```
for NG in $(aws eks list-nodegroups --cluster-name $MY_EKS_CLUSTER --query "nodegroups[*]" --output text)
do 
  aws eks delete-nodegroup --cluster $MY_EKS_CLUSTER --nodegroup-name $NG --no-page
done
 
for SERVICE in nodegroup
do
  for STACK in $(aws cloudformation list-stacks --region $MY_REGION --query "StackSummaries[?contains(StackName, '${SERVICE}')].{StackName:StackName}" --output text)
  do
    echo $STACK
    aws cloudformation delete-stack --region $MY_REGION --stack-name $STACK
  done
done
```

## Undeploy Java App (Pipeline)
## Delete ECR Repo
(need to figure out the code for this one)

## Delete all the stacks that were for the pipeline
```
for SERVICE in pipeline codecommit-ecr
do 
  for STACK in $(aws cloudformation list-stacks --region $MY_REGION --query "StackSummaries[?contains(StackName, '${SERVICE}')].{StackName:StackName}" --output text)
  do
    echo $STACK
    aws cloudformation delete-stack --region $MY_REGION --stack-name $STACK
  done
done
```
NOTE: You may need to doublecheck that there are no lingering ELB (I believe it remains from the helm deployment of the Java app - shouldbe interesting to see how to undo that correctly (rather than simply ripping the cluster away from the ELB)

## THIS ONE... SCORCH THE EARTH
```
for STACK in $(aws cloudformation list-stacks --region $MY_REGION --query "StackSummaries[?contains(StackStatus, 'CREATE_COMPLETE')].{StackName:StackName}" --output text)
do
  echo $STACK
  aws cloudformation delete-stack --region $MY_REGION --stack-name $STACK
done
```




