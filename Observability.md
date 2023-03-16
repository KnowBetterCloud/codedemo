# Observability
I can't believe how easy this is.

```
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=codedemo --approve
```

```
eksctl create iamserviceaccount \
  --cluster $MY_EKS_CLUSTER_NAME \
  --namespace amazon-cloudwatch \
  --name cloudwatch-agent \
  --attach-policy-arn  arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy \
  --override-existing-serviceaccounts \
  --approve
```

```
eksctl create iamserviceaccount \
  --cluster $MY_EKS_CLUSTER_NAME \
  --namespace amazon-cloudwatch \
  --name fluentd \
  --attach-policy-arn  arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy \
  --override-existing-serviceaccounts \
  --approve
```

```
curl -s https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluentd-quickstart.yaml | sed "s/{{cluster_name}}/$MY_EKS_CLUSTER_NAME/;s/{{region_name}}/${AWS_REGION}/" | kubectl apply -f -    
```

```
kubectl -n amazon-cloudwatch get daemonsets
```

```
eksctl create iamserviceaccount \
  --cluster $MY_EKS_CLUSTER_NAME \
  --namespace amazon-cloudwatch \
  --name cwagent-prometheus \
  --attach-policy-arn  arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy \
  --override-existing-serviceaccounts \
  --approve
```

```
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/service/cwagent-prometheus/prometheus-eks.yaml
```

```
kubectl get pod -l "app=cwagent-prometheus" -n amazon-cloudwatch
```


Enable Control-Plane Logs
```
eksctl utils update-cluster-logging \
    --enable-types all \
    --region ${AWS_REGION} \
    --cluster $MY_EKS_CLUSTER_NAME \
    --approve
```

Go to your AWS Console - CloudWatch | Insights | Container Insights  
