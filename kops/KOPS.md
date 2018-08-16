# Kops

Configuring Kops to work with Terraform

### Step 0 - Check permissions

Create an IAM user with these permissions
```
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
```

### Step 1 - Create a state storage bucket

`aws s3api create-bucket --bucket k8s-state-store --region us-east-1`

### Step 2 - Export some variables

```
# Gossip based cluster dns - https://github.com/kubernetes/kops/blob/master/docs/aws.md#configure-dns
export KOPS_NAME=cluster.k8s.local
export KOPS_STATE_STORE=s3://k8s-state-store
```

### Step 3 - Run Kops and export output to Terraform

```kops create cluster \
    --name $KOPS_NAME \
    --state $KOPS_STATE_STORE \
    --zones us-east-1a \
    --cloud aws \
    --master-size t2.medium \
    --node-size t2.medium \
    --out ./kops \
    --target terraform
```

### Step 4 - Terraform it!

`terraform plan`
`terraform apply`

### Step 5 - Workaround to be able to use kubectl with gossip-dns

```
kops export kubecfg $KOPS_NAME
kops update cluster $KOPS_NAME --target=terraform --out=.
terraform plan
terraform apply
kops rolling-update cluster $KOPS_NAME --cloudonly --force --yes
```

`kubectl get nodes` should work now.

### Destroy everything!

`terraform destroy`