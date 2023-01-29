# Deploy Kubernetes Dashboard

```
export DASHBOARD_VERSION="v2.6.0"

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/${DASHBOARD_VERSION}/aio/deploy/recommended.yaml

kubectl proxy --port=8080 --address=0.0.0.0 --disable-filter=true &
```


