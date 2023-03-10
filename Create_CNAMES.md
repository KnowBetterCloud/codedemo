# Create CNAME for App

STATUS:  

* 2023-01-23 - I *think* this should be complete/working now 
         If you find that I am missing something, please let me know.
* 2023-01-21 - Work in progress 
  Just realized I need to clean up the way I create the vars and what 
           echo one is used for.

## Visualization Demo App
Assuming the following is already exported as ENV variables  
```
MY_PROJECT=codedemo  
MY_PARENT_DOMAIN=clouditoutloud.com  
MY_PROJECT_DOMAIN=$MY_PROJECT.$MY_PARENT_DOMAIN
MY_APP_HOSTNAME="$MY_APP_NAME.$MY_PROJECT_DOMAIN"  
```

Get the Hosted Zone Id for the Project Domain
```
#export HOSTED_ZONE_ID=$(aws route53 list-hosted-zones-by-name |  jq --arg name "$MY_PROJECT_DOMAIN." -r '.HostedZones | .[] | select(.Name=="\($MY_PROJECT_DOMAIN)") | .Id' )
export HOSTED_ZONE_ID=$(aws route53 list-hosted-zones-by-name --dns-name '$MY_PROJECT_DOMAIN' --query "HostedZones[].Id" --output text)

# Hard coded with domain
# aws route53 list-hosted-zones-by-name --dns-name ${MY_PROJECT_DOMAIN} --query "HostedZones[].Id" --output text
# export HOSTED_ZONE_ID=$(aws route53 list-hosted-zones-by-name --dns-name codedemo.clouditoutloud.com. --query "HostedZones[].Id" --output text)
```

Pull down the CNAME template
```
curl -o cname_template.json https://raw.githubusercontent.com/KnowBetterCloud/codedemo/main/Files/cname_template.json
```

## Create CNAME for Visualization Load Balancer
* Get the hostname for the ELB
* replace values in the json template
* create cname
```
export MY_APP_NAME=ecsdemo-frontend 
export APP_ELB=$(kubectl get svc $MY_APP_NAME -o json | jq .status.loadBalancer.ingress[0].hostname | sed 's/"//g') 
echo -e "Adding \n CNAME: $MY_APP_NAME \n to DOMAIN: $MY_PROJECT_DOMAIN \n for ELB: $APP_ELB \n New Record: $MY_APP_NAME.$MY_PROJECT_DOMAIN"
envsubst < cname_template.json > cname_$MY_APP_NAME.json
aws route53 change-resource-record-sets --hosted-zone-id $HOSTED_ZONE_ID --change-batch file://cname_$MY_APP_NAME.json 
echo "Web Page for App: http://$MY_APP_NAME.$MY_PROJECT_DOMAIN/"
```

## Create NAME for Java App Load Balancer
```
export MY_APP_NAME=aws-proserve-java-greeting-dev
export APP_ELB=$(kubectl get svc $MY_APP_NAME -o json | jq .status.loadBalancer.ingress[0].hostname | sed 's/"//g') 
echo -e "Adding \n CNAME: $MY_APP_NAME \n to DOMAIN: $MY_PROJECT_DOMAIN \n for ELB: $APP_ELB \n New Record: $MY_APP_NAME.$MY_PROJECT_DOMAIN"
envsubst < cname_template.json > cname_$MY_APP_NAME.json
aws route53 change-resource-record-sets --hosted-zone-id $HOSTED_ZONE_ID --change-batch file://cname_$MY_APP_NAME.json 
echo "Web Page for App: http://$MY_APP_NAME.$MY_PROJECT_DOMAIN/hello/"
```
