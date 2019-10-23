# GKE with Terraform

## Requirements

* `terraform`
* `kubectl`
* `helm`
* `doctl`
* DigitalOcean account with admin privileges

## Create a Kubernetes cluster

```bash
terraform init

doctl kubernetes options versions

# TODO: Change token to a file

# Replace `[...]` with one of the older versions using a value from the `Slug` field.
terraform apply --var k8s_version=[...]

# NOTE: time elapsed: 3m33s, 5m52s

# NOTE: ClusterAutoscaler was implemented in early October 2019 and is still not available in Terraform

# NOTE: There is no regional cluster and, even if there would be, there are no regions with three zones.

doctl kubernetes cluster \
    kubeconfig save devops-paradox

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
    --set replicaCount=15 \
    --set image.repository=vfarcic/devops-toolkit-series \
    --set image.tag=latest \
    --set domain=false

kubectl apply \
    --filename devops-toolkit \
    --recursive

cd ..

kubectl get nodes

kubectl get pods

# Replace `[...]` with the same version you used before
terraform apply --var k8s_version=[...] --var node_count=3

kubectl get pods

kubectl get nodes
```

## Upgrade the cluster with a newer Kubernetes version

```bash
kubectl version --output yaml

doctl kubernetes options versions

# Replace `[...]` with the newest versioon using a value from the `Slug` field.
terraform apply --var k8s_version=[...] --var node_count=3

# TODO: time elapsed

kubectl get pods

kubectl version --output yaml

kubectl get nodes

kubectl get nodes
```

## Destroy the cluster

```bash
doctl kubernetes cluster \
    kubeconfig remove devops-paradox

terraform destroy
```