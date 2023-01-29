# Autoscaling


There are primarily two types of Autoscaling we will review  

* Cluster Autoscaling - scale the worker nodes to align with workload to allow the pods to run efficiently
* Pod Autoscaling - scale the number of pods needed to run your application


## Install Kube-Ops-View

```
helm install kube-ops-view \
stable/kube-ops-view \
--set service.type=LoadBalancer \
--set rbac.create=True
```

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml
kubectl get apiservice v1beta1.metrics.k8s.io -o json | jq '.status'
```

```
kubectl create deployment php-apache --image=us.gcr.io/k8s-artifacts-prod/hpa-example
kubectl set resources deploy php-apache --requests=cpu=200m
kubectl expose deploy php-apache --port 80
kubectl get pod -l app=php-apache
```

```
kubectl autoscale deployment php-apache `#The target average CPU utilization` \
    --cpu-percent=50 \
    --min=1 `#The lower limit for the number of pods that can be set by the autoscaler` \
    --max=10 `#The upper limit for the number of pods that can be set by the autoscaler`
```


Past this point is the "repeatable" or "demo" portion
Terminal 1
```
kubectl top node
kubectl get hpa -w
```

Terminal 2
```
kubectl run -i --tty load-generator --image=busybox /bin/sh
while true; do wget -q -O - http://php-apache; done
```

Terminal 1 (review the hpa command)
```
kubectl get hpa -w
```

Terminal 2
```
# Use CTRL-C to stop and CTRL-D to exit
kubectl delete pod load-generator
while true; do kubectl get pods | egrep 'load|php'; echo; sleep 2; done
```

# If things got out of hand.... (but.. HPA should handle things fine)
```
kubectl get rs
kubectl scale --replicas=1 rs/php-apache-6f8f979d4
```
