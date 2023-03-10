# Modify IAM Settings for your Workspace

We need to disable the dynamic credentials management that Cloud9 employs, and instead rely on the IAM role we created.

```
aws cloud9 update-environment --region=$MY_REGION  --environment-id $C9_PID --managed-credentials-action DISABLE
rm -vf ${HOME}/.aws/credentials
# Test access still works
aws s3 ls

export ACCOUNT_ID=$(aws sts get-caller-identity --region=$MY_REGION --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
export AZS=($(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text --region $AWS_REGION))

echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
echo "export AZS=(${AZS[@]})" | tee -a ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region

# If you did not source the variables file, this will fail (need to set MY_PROJECT=codedemo)
aws sts get-caller-identity --region=$MY_REGION --query Arn | grep $MY_PROJECT -q && echo "IAM role valid" || echo "IAM role NOT valid"
```

