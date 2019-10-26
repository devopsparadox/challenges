# Using Terraform To Install and Maintain A DigitalOcean Kubernetes (DOKS) Cluster

## Requirements

* `terraform`
* `kubectl`
* `helm`
* `doctl`
* DigitalOcean account with admin privileges

TODO: Gist

## Major Contributors

```bash
open "https://github.com/terraform-providers/terraform-provider-digitalocean/graphs/contributors"
```

* https://github.com/andrewsomething (DigitalOcean)
* https://github.com/stack72 (Pulumi)
* https://github.com/TFaga (OpenCredo)
* https://github.com/pearkes (HashiCorp)
* https://github.com/slapula (Oddball)

## Create a Kubernetes cluster

```bash
git clone https://github.com/vfarcic/k8s-specs.git

cd k8s-specs/terraform/doks

doctl compute region list
```

```
Slug    Name               Available
nyc1    New York 1         true
sgp1    Singapore 1        true
lon1    London 1           true
nyc3    New York 3         true
ams3    Amsterdam 3        true
fra1    Frankfurt 1        true
tor1    Toronto 1          true
sfo2    San Francisco 2    true
blr1    Bangalore 1        true
```

```bash
cat cluster.tf
```

```ruby
variable "cluster_name" {
  type    = string
  default = "devops-paradox"
}

variable "region" {
  type    = string
  default = "nyc1"
}

variable "machine_type" {
  type    = string
  default = "s-1vcpu-2gb"
}

variable "node_count" {
  type    = number
  default = 3
}

variable "k8s_version" {
  type = string
}

provider "digitalocean" {
  token = "${file("token")}"
}

resource "digitalocean_kubernetes_cluster" "primary" {
  name    = var.cluster_name
  region  = var.region
  version = var.k8s_version

  node_pool {
    name       = var.cluster_name
    size       = var.machine_type
    node_count = var.node_count
  }
}

output "cluster_name" {
  value = var.cluster_name
}

output "region" {
  value = var.region
}
```

```bash
# NOTE: ClusterAutoscaler was implemented in early October 2019 and, at the time of this writing (the end of October 2019) it is still not available in Terraform.

# NOTE: There is no regional cluster and, even if there would be, there are no regions with three zones.

terraform init

# Create the access token and store it in the file `token` inside this directory. Use [How to Create a Personal Access Token](https://www.digitalocean.com/docs/api/create-personal-access-token/) for instructions.

doctl auth init \
    --access-token $(cat token)
```

```
Using token [...]

Validating token... OK
```

```bash
doctl kubernetes options versions
```

```
Slug            Kubernetes Version
1.15.5-do.0     1.15.5
1.14.8-do.0     1.14.8
1.13.12-do.0    1.13.12
```

```bash
# Replace `[...]` with the second to latest version using the value from the `Slug` field.
export VERSION=[...]

terraform apply \
    --var k8s_version=$VERSION
```

```
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_kubernetes_cluster.primary will be created
  + resource "digitalocean_kubernetes_cluster" "primary" {
      + cluster_subnet = (known after apply)
      + created_at     = (known after apply)
      + endpoint       = (known after apply)
      + id             = (known after apply)
      + ipv4_address   = (known after apply)
      + kube_config    = (sensitive value)
      + name           = "devops-paradox"
      + region         = "nyc1"
      + service_subnet = (known after apply)
      + status         = (known after apply)
      + updated_at     = (known after apply)
      + version        = "1.14.8-do.0"

      + node_pool {
          + id         = (known after apply)
          + name       = "devops-paradox"
          + node_count = 3
          + nodes      = (known after apply)
          + size       = "s-1vcpu-2gb"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: 
```

```bash
# Type `yes` and press the enter key
```

```
digitalocean_kubernetes_cluster.primary: Creating...
digitalocean_kubernetes_cluster.primary: Still creating... [10s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [20s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [30s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [40s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [50s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [1m0s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [1m10s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [1m20s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [1m30s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [1m40s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [1m50s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [2m0s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [2m10s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [2m20s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [2m30s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [2m40s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [2m50s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [3m0s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [3m10s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [3m20s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [3m30s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [3m40s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [3m50s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [4m0s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [4m10s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [4m20s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [4m30s elapsed]
digitalocean_kubernetes_cluster.primary: Still creating... [4m40s elapsed]
digitalocean_kubernetes_cluster.primary: Creation complete after 4m43s [id=30586786-e221-4e6a-a096-40e200c32c63]
```

```bash
export KUBECONFIG=$PWD/kubeconfig

doctl kubernetes cluster \
    kubeconfig save devops-paradox
```

```
Notice: adding cluster credentials to kubeconfig file found in "/Users/vfarcic/.kube/config"
Notice: setting current-context to do-nyc1-devops-paradox
```

```bash
# NOTE: The first line of the output is incorrect

kubectl get nodes
```

```
NAME                  STATUS   ROLES    AGE     VERSION
devops-paradox-gewa   Ready    <none>   9m22s   v1.14.8
devops-paradox-gewe   Ready    <none>   9m10s   v1.14.8
devops-paradox-gewg   Ready    <none>   9m42s   v1.14.8
```

```bash
kubectl top nodes
```

```
Error from server (NotFound): the server could not find the requested resource (get services http:heapster:)
```

```bash
# NOTE: [metrics-server](https://github.com/kubernetes-incubator/metrics-server) is not installed by default
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
```

```
wrote /Users/vfarcic/code/k8s-specs/terraform/doks/devops-toolkit/devops-toolkit/templates/service.yaml
wrote /Users/vfarcic/code/k8s-specs/terraform/doks/devops-toolkit/devops-toolkit/templates/deployment.yaml
```

```bash
kubectl apply \
    --filename devops-toolkit \
    --recursive
```

```
deployment.extensions/devops-toolkit-devops-toolkit created
service/devops-toolkit created
```

```bash
cd ..

kubectl get pods
```

```
NAME                                             READY   STATUS    RESTARTS   AGE
devops-toolkit-devops-toolkit-86cf5f9f44-6zvgv   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-74qxx   0/1     Pending   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-7fzrl   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-bndsg   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-c8x4v   0/1     Pending   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-fcjcf   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-fgxbd   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-h6hcm   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-hvztq   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-jw9zh   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-kjm55   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-kn5ml   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-mbfjx   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-nd255   0/1     Pending   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-pq668   0/1     Pending   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-q4qdf   0/1     Pending   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-q5qpp   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-qw9m4   0/1     Pending   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-rmcgl   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-sdmjf   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-ssz89   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-vfqrd   0/1     Pending   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-vtwtx   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-x559q   1/1     Running   0          30s
devops-toolkit-devops-toolkit-86cf5f9f44-zpnn5   1/1     Running   0          30s
```

```bash
terraform apply \
    --var k8s_version=$VERSION \
    --var node_count=6
```

```
digitalocean_kubernetes_cluster.primary: Refreshing state... [id=30586786-e221-4e6a-a096-40e200c32c63]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # digitalocean_kubernetes_cluster.primary will be updated in-place
  ~ resource "digitalocean_kubernetes_cluster" "primary" {
        cluster_subnet = "10.244.0.0/16"
        created_at     = "2019-10-26 08:13:08 +0000 UTC"
        endpoint       = "https://30586786-e221-4e6a-a096-40e200c32c63.k8s.ondigitalocean.com"
        id             = "30586786-e221-4e6a-a096-40e200c32c63"
        ipv4_address   = "142.93.7.23"
        kube_config    = (sensitive value)
        name           = "devops-paradox"
        region         = "nyc1"
        service_subnet = "10.245.0.0/16"
        status         = "running"
        tags           = []
        updated_at     = "2019-10-26 08:17:47 +0000 UTC"
        version        = "1.14.8-do.0"

      ~ node_pool {
            id         = "d3b3cb0e-7a4b-42cd-8761-2a4f20056aba"
            name       = "devops-paradox"
          ~ node_count = 3 -> 6
            nodes      = [
                {
                    created_at = "2019-10-26 08:13:08 +0000 UTC"
                    id         = "ce554548-98c2-4f9b-aaa9-221f4643a5c4"
                    name       = "devops-paradox-gewg"
                    status     = "running"
                    updated_at = "2019-10-26 08:17:07 +0000 UTC"
                },
                {
                    created_at = "2019-10-26 08:13:08 +0000 UTC"
                    id         = "1799377e-b3d7-4ba2-89e5-e83992895e97"
                    name       = "devops-paradox-gewe"
                    status     = "running"
                    updated_at = "2019-10-26 08:17:07 +0000 UTC"
                },
                {
                    created_at = "2019-10-26 08:13:08 +0000 UTC"
                    id         = "ed29b41c-592d-4be2-881e-13b27f15d0a6"
                    name       = "devops-paradox-gewa"
                    status     = "running"
                    updated_at = "2019-10-26 08:17:07 +0000 UTC"
                },
            ]
            size       = "s-1vcpu-2gb"
            tags       = []
        }
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: 
```

```bash
# Type `yes` and press the enter key
```

```
digitalocean_kubernetes_cluster.primary: Modifying... [id=30586786-e221-4e6a-a096-40e200c32c63]
digitalocean_kubernetes_cluster.primary: Still modifying... [id=30586786-e221-4e6a-a096-40e200c32c63, 10s elapsed]
digitalocean_kubernetes_cluster.primary: Still modifying... [id=30586786-e221-4e6a-a096-40e200c32c63, 20s elapsed]
digitalocean_kubernetes_cluster.primary: Still modifying... [id=30586786-e221-4e6a-a096-40e200c32c63, 30s elapsed]
digitalocean_kubernetes_cluster.primary: Still modifying... [id=30586786-e221-4e6a-a096-40e200c32c63, 40s elapsed]
digitalocean_kubernetes_cluster.primary: Still modifying... [id=30586786-e221-4e6a-a096-40e200c32c63, 50s elapsed]
digitalocean_kubernetes_cluster.primary: Still modifying... [id=30586786-e221-4e6a-a096-40e200c32c63, 1m0s elapsed]
digitalocean_kubernetes_cluster.primary: Still modifying... [id=30586786-e221-4e6a-a096-40e200c32c63, 1m10s elapsed]
digitalocean_kubernetes_cluster.primary: Still modifying... [id=30586786-e221-4e6a-a096-40e200c32c63, 1m20s elapsed]
digitalocean_kubernetes_cluster.primary: Still modifying... [id=30586786-e221-4e6a-a096-40e200c32c63, 1m30s elapsed]
digitalocean_kubernetes_cluster.primary: Modifications complete after 1m32s [id=30586786-e221-4e6a-a096-40e200c32c63]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.

Outputs:

cluster_name = devops-paradox
region = nyc1
```

```bash
kubectl get pods
```

```
NAME                                             READY   STATUS    RESTARTS   AGE
devops-toolkit-devops-toolkit-86cf5f9f44-6zvgv   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-74qxx   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-7fzrl   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-bndsg   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-c8x4v   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-fcjcf   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-fgxbd   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-h6hcm   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-hvztq   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-jw9zh   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-kjm55   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-kn5ml   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-mbfjx   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-nd255   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-pq668   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-q4qdf   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-q5qpp   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-qw9m4   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-rmcgl   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-sdmjf   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-ssz89   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-vfqrd   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-vtwtx   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-x559q   1/1     Running   0          4m45s
devops-toolkit-devops-toolkit-86cf5f9f44-zpnn5   1/1     Running   0          4m45s
```

```bash
kubectl get nodes
```

```
NAME                  STATUS   ROLES    AGE   VERSION
devops-paradox-geg7   Ready    <none>   76s   v1.14.8
devops-paradox-gegc   Ready    <none>   82s   v1.14.8
devops-paradox-gegu   Ready    <none>   84s   v1.14.8
devops-paradox-gewa   Ready    <none>   17m   v1.14.8
devops-paradox-gewe   Ready    <none>   17m   v1.14.8
devops-paradox-gewg   Ready    <none>   17m   v1.14.8
```

## Upgrade the cluster with a newer Kubernetes version

```bash
kubectl version --output yaml
```

```yaml
clientVersion:
  buildDate: "2019-10-02T23:49:20Z"
  compiler: gc
  gitCommit: d647ddbd755faf07169599a625faf302ffc34458
  gitTreeState: clean
  gitVersion: v1.16.1
  goVersion: go1.12.9
  major: "1"
  minor: "16"
  platform: darwin/amd64
serverVersion:
  buildDate: "2019-10-15T12:02:12Z"
  compiler: gc
  gitCommit: 211047e9a1922595eaa3a1127ed365e9299a6c23
  gitTreeState: clean
  gitVersion: v1.14.8
  goVersion: go1.12.10
  major: "1"
  minor: "14"
  platform: linux/amd64
```

```bash
doctl kubernetes options versions

# Replace `[...]` with the newest version using a value from the `Slug` field.
export VERSION=[...]

# Open a second terminal session

export KUBECONFIG=$PWD/kubeconfig

# If MacOS
brew install watch

# Repeat periodically to confirm that rolling updates are used to upgrade nodes
watch --interval 10 \
    "kubectl get pods,nodes"

# Go back to the first terminal session

terraform apply \
    --var k8s_version=$VERSION \
    --var node_count=6
```

```
digitalocean_kubernetes_cluster.primary: Refreshing state... [id=30586786-e221-4e6a-a096-40e200c32c63]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # digitalocean_kubernetes_cluster.primary must be replaced
-/+ resource "digitalocean_kubernetes_cluster" "primary" {
      ~ cluster_subnet = "10.244.0.0/16" -> (known after apply)
      ~ created_at     = "2019-10-26 08:13:08 +0000 UTC" -> (known after apply)
      ~ endpoint       = "https://30586786-e221-4e6a-a096-40e200c32c63.k8s.ondigitalocean.com" -> (known after apply)
      ~ id             = "30586786-e221-4e6a-a096-40e200c32c63" -> (known after apply)
      ~ ipv4_address   = "142.93.7.23" -> (known after apply)
      ~ kube_config    = (sensitive value)
        name           = "devops-paradox"
        region         = "nyc1"
      ~ service_subnet = "10.245.0.0/16" -> (known after apply)
      ~ status         = "running" -> (known after apply)
      - tags           = [] -> null
      ~ updated_at     = "2019-10-26 08:33:48 +0000 UTC" -> (known after apply)
      ~ version        = "1.14.8-do.0" -> "1.15.5-do.0" # forces replacement

      ~ node_pool {
          ~ id         = "d3b3cb0e-7a4b-42cd-8761-2a4f20056aba" -> (known after apply)
            name       = "devops-paradox"
            node_count = 6
          ~ nodes      = [
              - {
                  - created_at = "2019-10-26 08:13:08 +0000 UTC"
                  - id         = "ce554548-98c2-4f9b-aaa9-221f4643a5c4"
                  - name       = "devops-paradox-gewg"
                  - status     = "running"
                  - updated_at = "2019-10-26 08:17:07 +0000 UTC"
                },
              - {
                  - created_at = "2019-10-26 08:13:08 +0000 UTC"
                  - id         = "1799377e-b3d7-4ba2-89e5-e83992895e97"
                  - name       = "devops-paradox-gewe"
                  - status     = "running"
                  - updated_at = "2019-10-26 08:17:07 +0000 UTC"
                },
              - {
                  - created_at = "2019-10-26 08:13:08 +0000 UTC"
                  - id         = "ed29b41c-592d-4be2-881e-13b27f15d0a6"
                  - name       = "devops-paradox-gewa"
                  - status     = "running"
                  - updated_at = "2019-10-26 08:17:07 +0000 UTC"
                },
              - {
                  - created_at = "2019-10-26 08:31:48 +0000 UTC"
                  - id         = "9983ae25-03ea-4fac-a1c9-fe25e5400c25"
                  - name       = "devops-paradox-gegu"
                  - status     = "running"
                  - updated_at = "2019-10-26 08:33:08 +0000 UTC"
                },
              - {
                  - created_at = "2019-10-26 08:31:48 +0000 UTC"
                  - id         = "69b53182-5080-417b-bf55-1af214f32b68"
                  - name       = "devops-paradox-gegc"
                  - status     = "running"
                  - updated_at = "2019-10-26 08:33:08 +0000 UTC"
                },
              - {
                  - created_at = "2019-10-26 08:31:48 +0000 UTC"
                  - id         = "36a19dcf-1e06-46bd-9551-da02d2f0b64e"
                  - name       = "devops-paradox-geg7"
                  - status     = "running"
                  - updated_at = "2019-10-26 08:33:08 +0000 UTC"
                },
            ] -> (known after apply)
            size       = "s-1vcpu-2gb"
          - tags       = [] -> null
        }
    }

Plan: 1 to add, 0 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: 
```

```bash
# Type `no` to cancel it
```

```
Apply cancelled.
```

```bash
doctl kubernetes cluster list
```

```
ID                                      Name              Region    Version        Auto Upgrade    Status     Node Pools
30586786-e221-4e6a-a096-40e200c32c63    devops-paradox    nyc1      1.14.8-do.0    false           running    devops-paradox
```

```bash
# Replace `[...]` with the ID (e.g., `30586786-e221-4e6a-a096-40e200c32c63`)
export CLUSTER_ID=[...]

doctl kubernetes cluster \
    upgrade $CLUSTER_ID \
    --version $VERSION
```

```
Notice: upgrading cluster to version 1.15.5-do.0
```

```bash
doctl kubernetes cluster get $CLUSTER_ID
```

```
ID                                      Name              Region    Version        Auto Upgrade    Status       Endpoint                                                               IPv4           Cluster Subnet    Service Subnet    Tags                                            Created At                       Updated At                       Node Pools
30586786-e221-4e6a-a096-40e200c32c63    devops-paradox    nyc1      1.15.5-do.0    false           upgrading    https://30586786-e221-4e6a-a096-40e200c32c63.k8s.ondigitalocean.com    142.93.7.23    10.244.0.0/16     10.245.0.0/16     k8s,k8s:30586786-e221-4e6a-a096-40e200c32c63    2019-10-26 08:13:08 +0000 UTC    2019-10-26 08:46:57 +0000 UTC    devops-paradox
```

```bash
# NOTE: The output from the second terminal session
```

```
Every 10.0s: kubectl get pods,nodes                        Viktors-MacBook-Pro.local: Sat Oct 26 10:47:25 2019

Unable to connect to the server: dial tcp 142.93.7.23:443: connect: operation timed out
Unable to connect to the server: dial tcp 142.93.7.23:443: connect: operation timed out
```

```bash
# NOTE: The output from the second terminal session
```

```
Every 10.0s: kubectl get pods,nodes                        Viktors-MacBook-Pro.local: Sat Oct 26 10:51:24 2019

NAME                                                 READY   STATUS        RESTARTS   AGE
pod/devops-toolkit-devops-toolkit-86cf5f9f44-6zvgv   1/1     Terminating   0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-74qxx   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-7fzrl   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-bndsg   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-c8x4v   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-fcjcf   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-fgxbd   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-h6hcm   1/1     Terminating   0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-hvztq   1/1     Terminating   0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-jw9zh   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-kjm55   1/1     Terminating   0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-kn5ml   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-mbfjx   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-nd255   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-pq668   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-q4qdf   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-q5qpp   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-qw9m4   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-rmcgl   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-sdmjf   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-ssz89   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-vfqrd   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-vtwtx   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-x559q   1/1     Running       0          21m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-zpnn5   1/1     Running       0          21m

NAME                       STATUS                     ROLES    AGE   VERSION
node/devops-paradox-geg7   Ready                      <none>   18m   v1.14.8
node/devops-paradox-gegc   Ready                      <none>   18m   v1.14.8
node/devops-paradox-gegu   Ready                      <none>   18m   v1.14.8
node/devops-paradox-gewa   Ready                      <none>   34m   v1.14.8
node/devops-paradox-gewe   Ready                      <none>   33m   v1.14.8
node/devops-paradox-gewg   Ready,SchedulingDisabled   <none>   34m   v1.14.8
```

```bash
doctl kubernetes cluster get $CLUSTER_ID
```

```
ID                                      Name              Region    Version        Auto Upgrade    Status     Endpoint                                                               IPv4             Cluster Subnet    Service Subnet    Tags                                            Created At                       Updated At                       Node Pools
30586786-e221-4e6a-a096-40e200c32c63    devops-paradox    nyc1      1.15.5-do.0    false           running    https://30586786-e221-4e6a-a096-40e200c32c63.k8s.ondigitalocean.com    142.93.14.150    10.244.0.0/16     10.245.0.0/16     k8s,k8s:30586786-e221-4e6a-a096-40e200c32c63    2019-10-26 08:13:08 +0000 UTC    2019-10-26 08:51:47 +0000 UTC    devops-paradox
```

```bash
# NOTE: The output from the second terminal session
```

```
Every 10.0s: kubectl get pods,nodes                        Viktors-MacBook-Pro.local: Sat Oct 26 10:52:50 2019

NAME                                                 READY   STATUS              RESTARTS   AGE
pod/devops-toolkit-devops-toolkit-86cf5f9f44-5njtf   0/1     ContainerCreating   0          49s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-74qxx   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-7fzrl   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-bndsg   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-c8w5j   0/1     ContainerCreating   0          49s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-c8x4v   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-cqblv   0/1     ContainerCreating   0          49s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-fcjcf   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-fgxbd   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-jw9zh   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-kn5ml   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-lbdl5   0/1     ContainerCreating   0          48s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-mbfjx   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-nd255   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-pq668   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-q4qdf   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-q5qpp   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-qw9m4   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-rmcgl   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-sdmjf   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-ssz89   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-vfqrd   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-vtwtx   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-x559q   1/1     Running             0          23m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-zpnn5   1/1     Running             0          23m

NAME                       STATUS                     ROLES    AGE   VERSION
node/devops-paradox-geg7   Ready                      <none>   19m   v1.14.8
node/devops-paradox-gegc   Ready                      <none>   19m   v1.14.8
node/devops-paradox-gegu   Ready                      <none>   19m   v1.14.8
node/devops-paradox-gewa   Ready                      <none>   35m   v1.14.8
node/devops-paradox-gewe   Ready                      <none>   35m   v1.14.8
node/devops-paradox-gewg   Ready,SchedulingDisabled   <none>   35m   v1.14.8
```

```bash
# NOTE: The output from the second terminal session
```

```
Every 10.0s: kubectl get pods,nodes                        Viktors-MacBook-Pro.local: Sat Oct 26 10:57:29 2019

NAME                                                 READY   STATUS              RESTARTS   AGE
pod/devops-toolkit-devops-toolkit-86cf5f9f44-5kwvn   0/1     Running             0          9s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-5njtf   1/1     Running             0          5m28s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-74qxx   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-9s8fp   0/1     ContainerCreating   0          10s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-c8w5j   1/1     Running             0          5m28s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-c8x4v   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-cqblv   1/1     Running             0          5m28s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-dlh8s   1/1     Running             0          10s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-fcjcf   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-jw9zh   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-k7xfg   0/1     Running             0          10s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-kn5ml   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-lbdl5   1/1     Running             0          5m27s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-mbfjx   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-nd255   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-npbn6   0/1     ContainerCreating   0          10s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-pq668   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-q4qdf   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-q5qpp   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-qw9m4   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-sdmjf   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-t9vjw   0/1     ContainerCreating   0          10s
pod/devops-toolkit-devops-toolkit-86cf5f9f44-vfqrd   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-x559q   1/1     Running             0          27m
pod/devops-toolkit-devops-toolkit-86cf5f9f44-zgj6g   0/1     ContainerCreating   0          10s

NAME                       STATUS                     ROLES    AGE   VERSION
node/devops-paradox-geen   Ready                      <none>   55s   v1.15.5
node/devops-paradox-geg7   Ready                      <none>   24m   v1.14.8
node/devops-paradox-gegc   Ready                      <none>   24m   v1.14.8
node/devops-paradox-gegu   Ready                      <none>   24m   v1.14.8
node/devops-paradox-gewa   Ready                      <none>   40m   v1.14.8
node/devops-paradox-gewe   Ready,SchedulingDisabled   <none>   40m   v1.14.8
```

```bash
# Wait until all the nodes are upgraded by looking at the second terminal session

kubectl version --output yaml
```

```
clientVersion:
  buildDate: "2019-10-02T23:49:20Z"
  compiler: gc
  gitCommit: d647ddbd755faf07169599a625faf302ffc34458
  gitTreeState: clean
  gitVersion: v1.16.1
  goVersion: go1.12.9
  major: "1"
  minor: "16"
  platform: darwin/amd64
serverVersion:
  buildDate: "2019-10-15T19:07:57Z"
  compiler: gc
  gitCommit: 20c265fef0741dd71a66480e35bd69f18351daea
  gitTreeState: clean
  gitVersion: v1.15.5
  goVersion: go1.12.10
  major: "1"
  minor: "15"
  platform: linux/amd64
```

## Pros And Cons

**Pros**

TODO:

* DigitalOcean is contributing to Terraform
* Terraform definition is easy and straightforward
* It's fast
* Rolling upgrade is simple with `doctl`

**Cons**

TODO:

* ClusterAutoscaler was implemented in early October 2019 and, at the time of this writing (the end of October 2019) it is still not available in Terraform.
* There is no region with three zones and there is no option to create a regional cluster
* [metrics-server](https://github.com/kubernetes-incubator/metrics-server) is not installed by default
* Upgrade with Terraform destroys the whole cluster
* Upgrade with `doctl` produces downtime since there is only one master
* Upgrade with `doctl` destroys a node before creating a new one

**The Checklist**

- [ ] Defined as code and stored in Git
- [ ] Idempotent
- [ ] Done through Terraform
- [ ] Speed
- [ ] Easy installation and maintenance
- [ ] Fault tollerant
- [ ] Highly-available
- [ ] Can be scaled automatically
- [ ] Can be upgraded
- [ ] Personal preference

## Destroy the cluster

```bash
terraform destroy \
    --var k8s_version=$VERSION
```

```
digitalocean_kubernetes_cluster.primary: Refreshing state... [id=30586786-e221-4e6a-a096-40e200c32c63]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # digitalocean_kubernetes_cluster.primary will be destroyed
  - resource "digitalocean_kubernetes_cluster" "primary" {
      - cluster_subnet = "10.244.0.0/16" -> null
      - created_at     = "2019-10-26 08:13:08 +0000 UTC" -> null
      - endpoint       = "https://30586786-e221-4e6a-a096-40e200c32c63.k8s.ondigitalocean.com" -> null
      - id             = "30586786-e221-4e6a-a096-40e200c32c63" -> null
      - ipv4_address   = "142.93.14.150" -> null
      - kube_config    = (sensitive value)
      - name           = "devops-paradox" -> null
      - region         = "nyc1" -> null
      - service_subnet = "10.245.0.0/16" -> null
      - status         = "running" -> null
      - tags           = [] -> null
      - updated_at     = "2019-10-26 09:04:57 +0000 UTC" -> null
      - version        = "1.15.5-do.0" -> null

      - node_pool {
          - id         = "d3b3cb0e-7a4b-42cd-8761-2a4f20056aba" -> null
          - name       = "devops-paradox" -> null
          - node_count = 6 -> null
          - nodes      = [
              - {
                  - created_at = "2019-10-26 08:31:48 +0000 UTC"
                  - id         = "9983ae25-03ea-4fac-a1c9-fe25e5400c25"
                  - name       = "devops-paradox-gegu"
                  - status     = "deleting"
                  - updated_at = "2019-10-26 09:04:49 +0000 UTC"
                },
              - {
                  - created_at = "2019-10-26 08:31:48 +0000 UTC"
                  - id         = "69b53182-5080-417b-bf55-1af214f32b68"
                  - name       = "devops-paradox-gegc"
                  - status     = "running"
                  - updated_at = "2019-10-26 08:33:08 +0000 UTC"
                },
              - {
                  - created_at = "2019-10-26 08:31:48 +0000 UTC"
                  - id         = "36a19dcf-1e06-46bd-9551-da02d2f0b64e"
                  - name       = "devops-paradox-geg7"
                  - status     = "running"
                  - updated_at = "2019-10-26 08:33:08 +0000 UTC"
                },
              - {
                  - created_at = "2019-10-26 08:54:29 +0000 UTC"
                  - id         = "56e76854-2b12-4075-9db0-312c0c159f25"
                  - name       = "devops-paradox-geen"
                  - status     = "running"
                  - updated_at = "2019-10-26 08:55:49 +0000 UTC"
                },
              - {
                  - created_at = "2019-10-26 08:59:19 +0000 UTC"
                  - id         = "0fd62024-fefe-4f9f-9ff8-64845fe5ade4"
                  - name       = "devops-paradox-geeu"
                  - status     = "running"
                  - updated_at = "2019-10-26 09:00:39 +0000 UTC"
                },
              - {
                  - created_at = "2019-10-26 09:03:30 +0000 UTC"
                  - id         = "b9c673bb-bb69-4b68-810d-9c85e7a71c81"
                  - name       = "devops-paradox-geeq"
                  - status     = "running"
                  - updated_at = "2019-10-26 09:04:49 +0000 UTC"
                },
            ] -> null
          - size       = "s-1vcpu-2gb" -> null
          - tags       = [] -> null
        }
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: 
```

```bash
# Type `yes` and press the enter key
```

```
digitalocean_kubernetes_cluster.primary: Destroying... [id=30586786-e221-4e6a-a096-40e200c32c63]
digitalocean_kubernetes_cluster.primary: Destruction complete after 1s

Destroy complete! Resources: 1 destroyed.
```