# GKE with Terraform

## Requirements

* `terraform`
* `kubectl`
* `helm`
* `gcloud`
* GCP account with admin privileges

## Major Contributors

```bash
open "https://github.com/terraform-providers/terraform-provider-google/graphs/contributors"
```

* https://github.com/modular-magician (?)
* https://github.com/danawillow (Google)
* https://github.com/rosbo (Google)
* https://github.com/rileykarson (Google)
* https://github.com/paddycarver (HashiCorp)

## Create a Kubernetes cluster

* [Create a service account](https://console.cloud.google.com/apis/credentials/serviceaccountkey), download the JSON, and store it as account.json in this directory

```bash
# TODO: List zones

cat cluster.tf

terraform init

gcloud auth login

gcloud container get-server-config

# Replace `[...]` with one of the older versions from the `validMasterVersions` section.
export VERSION=[...]

terraform apply \
    --var k8s_version=$VERSION

# Time elapsed: 6m40s+58s, 6m31s+1m1s

export KUBECONFIG=$PWD/kubeconfig

gcloud container clusters \
    get-credentials $(terraform output cluster_name) \
	--region $(terraform output region)

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
    --set replicaCount=25 \
    --set image.repository=vfarcic/devops-toolkit-series \
    --set image.tag=latest \
    --set domain=false

kubectl apply \
    --filename devops-toolkit \
    --recursive

SECONDS=0

cd ..

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

gcloud container get-server-config \
    --region $(terraform output region)

# Replace `[...]` with the newest version from the `validMasterVersions` section
export VERSION=[...]

terraform apply \
    --var k8s_version=$VERSION

# Open a second terminal session

# Repeat periodically to confirm that rolling updates are used to upgrade nodes
kubectl get pods,nodes

# Go back to the first terminal session

# Time elapsed: 16m0s+14m24ss

kubectl get pods

kubectl version --output yaml

kubectl get nodes
```

## Pros And Cons

**Pros**

TODO:

**Cons**

TODO:

**The Checklist**

TODO:

## Destroy the cluster

```bash
terraform destroy \
    --var k8s_version=$VERSION

# If it throws an error stating that the cluster is being upgraded, wait for a while and repeat the previous command
```