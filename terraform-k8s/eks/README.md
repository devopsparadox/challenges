# EKS with Terraform

## Requirements

* `terraform`
* `kubectl`
* `helm`
* `aws`
* AWS account with admin privileges

## Major Contributors

```bash
open "https://github.com/terraform-aws-modules/terraform-aws-eks/graphs/contributors"
```

* https://github.com/max-rocket-internet (Delivery Hero)
* https://github.com/brandoconnor (Run at Scale)
* https://github.com/jimbeck (BTI360)
* https://github.com/dpiddockcmp (?)
* https://github.com/barryib (Polyconseil)

## Create a Kubernetes cluster

* Create a VPC
* Create a subnet
* Create an internet gateway
* Create a route table
* Create a route table association
* Create an IAM role
* Create a few IAM role policies
* Create a security group
* Create a security group rule
* Create an EKS cluster (only master nodes)
* Add entries to kubeconfig

* Create an IAM role for worker nodes
* Create a few IAM role policies for worker nodes
* Create a security group for worker nodes
* Create a few security group rules for worker nodes
* Create a node launch configuration for auto-scaling groups
* Create an auto-scaling group
* Join worker nodes to the EKS cluster

NOTE: This is ridiculous!!! Abandoning Terraform and GitOps...

```bash
export AWS_ACCESS_KEY_ID=[...] # Replace [...] with the AWS Access Key ID

export AWS_SECRET_ACCESS_KEY=[...] # Replace [...] with the AWS Secret Access Key

export AWS_DEFAULT_REGION=us-west-2

eksctl create cluster --help \
    | grep version

# Replace `[...]` with one of the older versions.
export VERSION=[...]

export CLUSTER_NAME=devops-paradox

export KUBECONFIG=$PWD/kubeconfig

SECONDS=0

eksctl create cluster \
    --name $CLUSTER_NAME \
    --region $AWS_DEFAULT_REGION \
    --node-type t2.small \
    --nodes-max 6 \
    --nodes-min 3 \
    --asg-access \
    --version $VERSION

echo "$(($SECONDS / 60))m$(($SECONDS % 60))s"

# Time elapsed: 15m18s

IAM_ROLE=$(aws iam list-roles \
    | jq -r ".Roles[] \
    | select(.RoleName \
    | startswith(\"eksctl-$CLUSTER_NAME-nodegroup\")) \
    .RoleName")

echo $IAM_ROLE

aws iam put-role-policy \
    --role-name $IAM_ROLE \
    --policy-name $CLUSTER_NAME-AutoScaling \
    --policy-document https://raw.githubusercontent.com/vfarcic/k8s-specs/master/scaling/eks-autoscaling-policy.json

mkdir -p charts

helm fetch stable/cluster-autoscaler \
    -d charts \
    --untar

mkdir -p k8s-specs/aws

helm template charts/cluster-autoscaler \
    --name aws-cluster-autoscaler \
    --output-dir k8s-specs/aws \
    --namespace kube-system \
    --set autoDiscovery.clusterName=$CLUSTER_NAME \
    --set awsRegion=$AWS_DEFAULT_REGION \
    --set sslCertPath=/etc/kubernetes/pki/ca.crt \
    --set rbac.create=true

kubectl apply \
    -n kube-system \
    -f k8s-specs/aws/cluster-autoscaler/*

kubectl get nodes

kubectl top nodes
```

## Fill the cluster beyond its capacity and scale it to accomodate that

```bash
git clone \
    https://github.com/vfarcic/devops-toolkit.git

cd devops-toolkit

helm template charts/devops-toolkit \
    --name devops-toolkit \
    --output-dir $PWD \
    --set replicaCount=35 \
    --set image.repository=vfarcic/devops-toolkit-series \
    --set image.tag=latest \
    --set domain=false

kubectl apply \
    --filename devops-toolkit \
    --recursive

cd ..

SECONDS=0

kubectl get nodes

kubectl get pods

# Wait for a while and repeat the previous command if some Pods are in the `pending` state

echo "$(($SECONDS / 60))m$(($SECONDS % 60))s"

# Time elapsed: TODO:

kubectl get nodes
```

## Upgrade the cluster with a newer Kubernetes version

```bash
kubectl version --output yaml

eksctl create cluster --help \
    | grep version

# Replace `[...]` with the newest version
export VERSION=[...]

# Cannot update to a specific version

eksctl update cluster \
    --name=$CLUSTER_NAME

eksctl update cluster \
    --name=$CLUSTER_NAME \
    --approve

# Open a second terminal session

export AWS_ACCESS_KEY_ID=[...] # Replace [...] with the AWS Access Key ID

export AWS_SECRET_ACCESS_KEY=[...] # Replace [...] with the AWS Secret Access Key

export KUBECONFIG=$PWD/kubeconfig

# Repeat periodically to confirm that rolling updates are used to upgrade nodes
kubectl get pods,nodes

# Go back to the first terminal session

kubectl version --output yaml

eksctl get nodegroups \
    --cluster=$CLUSTER_NAME

NODE_GROUP=[...]

eksctl create nodegroup \
    --cluster $CLUSTER_NAME \
    --region $AWS_DEFAULT_REGION \
    --node-type t2.small \
    --nodes-max 6 \
    --nodes-min 3 \
    --asg-access \
    --version $VERSION    

kubectl get nodes

# aws iam delete-role-policy \
#     --role-name $IAM_ROLE \
#     --policy-name $CLUSTER_NAME-AutoScaling

# IAM_ROLE=$(aws iam list-roles \
#     | jq -r ".Roles[] \
#     | select(.RoleName \
#     | startswith(\"eksctl-$CLUSTER_NAME-nodegroup\")) \
#     .RoleName")

# echo $IAM_ROLE

# aws iam put-role-policy \
#     --role-name $IAM_ROLE \
#     --policy-name $CLUSTER_NAME-AutoScaling \
#     --policy-document https://raw.githubusercontent.com/vfarcic/k8s-specs/master/scaling/eks-autoscaling-policy.json

eksctl delete nodegroup \
    --cluster $CLUSTER_NAME \
    --name $NODE_GROUP

# Time elapsed: TODO

# Go back to the second terminal session

# Repeat periodically to confirm that rolling updates are used to upgrade nodes
kubectl get pods,nodes

# Go back to the first terminal session

kubectl get pods

kubectl version --output yaml

kubectl get nodes
```

## Destroy the cluster

```bash
IAM_ROLE=$(aws iam list-roles \
    | jq -r ".Roles[] \
    | select(.RoleName \
    | startswith(\"eksctl-$CLUSTER_NAME-nodegroup\")) \
    .RoleName")

echo $IAM_ROLE

aws iam delete-role-policy \
    --role-name $IAM_ROLE \
    --policy-name $CLUSTER_NAME-AutoScaling

eksctl delete cluster -n $CLUSTER_NAME

# Delete unused volumes
for volume in `aws ec2 describe-volumes --output text| grep available | awk '{print $8}'`; do 
    echo "Deleting volume $volume"
    aws ec2 delete-volume --volume-id $volume
done
```