# GKE with Terraform

## Requirements

* Terraform
* kubectl
* Helm
* GCP account with admin privileges

## Create a Kubernetes cluster

* [Create a service account](https://console.cloud.google.com/apis/credentials/serviceaccountkey), down the JSON, and store it as account.json in this directory

```bash
terraform init

gcloud container get-server-config

# Replace `[...]` with one of the older versions from the `validMasterVersions` section.
terraform apply --var k8s_version=[...]

gcloud container clusters \
    get-credentials $(terraform output cluster_name) \
	--region $(terraform output region)

kubectl get nodes
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

cd ..

kubectl get nodes

kubectl get pods

# Wait for a while and repeat the previous command if some Pods are in the `pending` state

kubectl get nodes
```

## Upgrade the cluster with a newer Kubernetes version

```bash
kubectl version --output yaml

gcloud container get-server-config \
    --region $(terraform output region)

# Replace `[...]` with the newest version from the `validMasterVersions` section
terraform apply --var k8s_version=[...]

kubectl get pods

kubectl get nodes

kubectl version --output yaml
```

## Destroy the cluster

```bash
terraform destroy

# TODO: Remove from KUBECONFIG
```