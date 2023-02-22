# 

# If you did not source the variables file, this will fail (need to set PROJECT=codedemo)
```
aws sts get-caller-identity --region $MY_REGION --query Arn | grep $PROJECT -q && echo "IAM role valid" || echo "IAM role NOT valid"

ROLE="    - rolearn: arn:aws:iam::${ACCOUNT_ID}:role/EksWorkshopCodeBuildKubectlRole\n      username: build\n      groups:\n        - system:masters"
kubectl get -n kube-system configmap/aws-auth -o yaml | awk "/mapRoles: \|/{print;print \"$ROLE\";next}1" > /tmp/aws-auth-patch.yml
kubectl patch configmap/aws-auth -n kube-system --patch "$(cat /tmp/aws-auth-patch.yml)"

