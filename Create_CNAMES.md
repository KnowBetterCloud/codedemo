# Create CNAME for App

STATUS:  Work in progress - 2023-01-21
         Just realized I need to clean up the way I create the vars and what 
           echo one is used for.

## Visualization Demo App
Assuming the following is already exported as ENV variables
APP_NAME=visualization
PROJECT=codedemo
PARENT_DOMAIN=clouditoutlout.com
PROJECT_DOMAIN=$PROJECT.$PARENT_DOMAIN
APP_HOSTNAME="$APP_NAME.$PROJECT_DOMAIN"  

Get the hostname for the Visualization Load Balancer
```
kubectl  get svc ecsdemo-frontend -o json | jq .status.loadBalancer.ingress[0].hostname
```
