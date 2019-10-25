# EKS with Terraform

## Requirements

* `terraform`
* `kubectl`
* `helm`
* `az`
* Azure account with admin privileges

## Major Contributors

```bash
open "https://github.com/terraform-providers/terraform-provider-azurerm/graphs/contributors"
```

TODO:

* https://github.com/tombuildsstuff (HashiCorp)
* https://github.com/katbyte (HashiCorp)
* https://github.com/mbfrahry (HashiCorp)
* https://github.com/stack72 (Pulumi)
* https://github.com/metacpp (Microsoft)

## Create a Kubernetes cluster

* Create a new service principal in [App registrations](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps), and store the *Application (client) ID as `client_id` file in this directory
* Create a *New client secret*, and store it as `client_secret` file in this directory

```bash
rm -rf *.tfstate*

cat cluster.tf

terraform init

az login

az aks get-versions \
    --location eastus \
    --output table 

# Replace `[...]` with one of the older versions from the `validMasterVersions` section.
export VERSION=[...]

terraform apply \
    --var k8s_version=$VERSION

# Time elapsed: 23m32s

export KUBECONFIG=$PWD/kubeconfig

az aks get-credentials \
    --name $(terraform output cluster_name) \
    --resource-group $(terraform output cluster_name)

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
    --set replicaCount=60 \
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

kubectl get nodes
```

## Upgrade the cluster with a newer Kubernetes version

```bash
kubectl version --output yaml

az aks get-versions \
    --location $(terraform output location) \
    --output table

# Replace `[...]` with the newest version from the `validMasterVersions` section
export VERSION=[...]

terraform apply \
    --var k8s_version=$VERSION

# Cancel

export VM_COUNT=$(kubectl get nodes \
    --no-headers \
    | wc -l \
    | tr -d "[:space:]")

terraform apply \
    --var k8s_version=$VERSION \
    --var vm_count=$VM_COUNT

terraform apply \
    --var k8s_version=$VERSION \
    --var vm_count=$VM_COUNT \
    --var auto_scaling=false

az aks update \
  --resource-group $(terraform output cluster_name) \
  --name $(terraform output cluster_name) \
  --disable-cluster-autoscaler

cat cluster.tf \
    | sed -e \
    's@  min_count@  # min_count@g' \
    | sed -e \
    's@  max_count@  # max_count@g' \
    | tee cluster.tf

terraform apply \
    --var k8s_version=$VERSION \
    --var vm_count=$VM_COUNT \
    --var auto_scaling=false

az aks update \
  --resource-group $(terraform output cluster_name) \
  --name $(terraform output cluster_name) \
  --enable-cluster-autoscaler \
  --min-count $(terraform output min_count) \
  --max-count $(terraform output max_count)

cat cluster.tf \
    | sed -e \
    's@  # min_count@  min_count@g' \
    | sed -e \
    's@  # max_count@  max_count@g' \
    | tee cluster.tf

terraform apply \
    --var k8s_version=$VERSION \
    --var vm_count=$VM_COUNT
```

## Destroy the cluster

```bash
terraform destroy \
    --var k8s_version=$VERSION

# TODO: Remove from KUBECONFIG
```