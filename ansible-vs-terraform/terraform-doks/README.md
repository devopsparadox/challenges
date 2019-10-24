# GKE with Terraform

## Requirements

* `terraform`
* `kubectl`
* `helm`
* `doctl`
* DigitalOcean account with admin privileges

## Create a Kubernetes cluster

```bash
cat cluster.tf

terraform init

# Create the access token and store it in the file `token` inside this directory.

# Replace `[...]` with your access token
doctl auth init \
    --access-token $(cat token)

# NOTE: ClusterAutoscaler was implemented in early October 2019 and is still not available in Terraform

doctl kubernetes options versions

# Replace `[...]` with one of the older versions using a value from the `Slug` field.
export VERSION=[...]

terraform apply \
    --var k8s_version=$VERSION

# Time elapsed: 3m33s, 5m52s, 4m11s, 5m52s

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

# Replace `[...]` with the same version you used before
terraform apply \
    --var k8s_version=$VERSION \
    --var node_count=6

# Time elapsed: 2m12s,2m11s

kubectl get pods

kubectl get nodes
```

## Upgrade the cluster with a newer Kubernetes version

```bash
kubectl version --output yaml

doctl kubernetes options versions

# Replace `[...]` with the newest version using a value from the `Slug` field.
export VERSION=[...]

# TODO: Figure out how to to do rolling upgrade

# Open a second terminal session

# Repeat periodically to confirm that rolling updates are used to upgrade nodes
kubectl get pods,nodes

# Go back to the first terminal session

terraform apply \
    --var k8s_version=$VERSION \
    --var node_count=6

# Cancel it

doctl kubernetes cluster list

# Replace `[...]` with the ID
export CLUSTER_ID=[...]

# TODO: Continue

doctl kubernetes cluster \
    upgrade $CLUSTER_ID \
    --version $VERSION

kubectl get nodes

# It fails until the master node is upgraded.

# Repeat the command until all the nodes are upgraded.

# TODO: time elapsed

kubectl get pods

kubectl version --output yaml

kubectl get nodes

terraform refresh \
    --var k8s_version=$VERSION \
    --var node_count=6

terraform apply \
    --var k8s_version=$VERSION \
    --var node_count=6
```

## Destroy the cluster

```bash
doctl kubernetes cluster \
    kubeconfig remove devops-paradox

terraform destroy \
    --var k8s_version=$VERSION
```