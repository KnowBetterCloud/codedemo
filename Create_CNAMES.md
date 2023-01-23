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
PROJECT=codedemo  
PARENT_DOMAIN=clouditoutlout.com  
PROJECT_DOMAIN=$PROJECT.$PARENT_DOMAIN
APP_HOSTNAME="$APP_NAME.$PROJECT_DOMAIN"  
```

Get the Hosted Zone Id for the Project Domain
```
export HOSTED_ZONE_ID=$(aws route53 list-hosted-zones-by-name |  jq --arg name "$PROJECT_DOMAIN." -r '.HostedZones | .[] | select(.Name=="\($name)") | .Id' )
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
export APP_NAME=ecsdemo-frontend 
export APP_ELB=$(kubectl get svc $APP_NAME -o json | jq .status.loadBalancer.ingress[0].hostname | sed 's/"//g') 
echo -e "Adding \n CNAME: $APP_NAME \n to DOMAIN: $PROJECT_DOMAIN \n for ELB: $APP_ELB \n New Record: $APP_NAME.$PROJECT_DOMAIN"
envsubst < cname_template.json > cname_$APP_NAME.json
aws route53 change-resource-record-sets --hosted-zone-id $HOSTED_ZONE_ID --change-batch file://cname_$APP_NAME.json 
hange-config.json
echo "Web Page for App: $APP_NAME.$PROJECT_DOMAIN/hello/"
```

## Create NAME for Java App Load Balancer
```
export APP_NAME=aws-proserve-java-greeting-dev
export APP_ELB=$(kubectl get svc $APP_NAME -o json | jq .status.loadBalancer.ingress[0].hostname | sed 's/"//g') 
echo -e "Adding \n CNAME: $APP_NAME \n to DOMAIN: $PROJECT_DOMAIN \n for ELB: $APP_ELB \n New Record: $APP_NAME.$PROJECT_DOMAIN"
aws route53 change-resource-record-sets --hosted-zone-id $HOSTED_ZONE_ID --change-batch file://cname_$APP_NAME.json 
```
