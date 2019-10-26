# Compare Managed Kubernetes Clusters

## Rules (when practical and possible)

* Should be defined as code and stored in Git
* Should be idempotent
* Should be done through Terraform

## Criterias

* Speed
* Ease of installation and maintenance
* Whether it is fault tollerant
* Whether it is highly-available
* Whether it can be scaled automatically
* Whether it can be upgraded

## What

* Create a fault-tollerant Kubernetes cluster
* Fill the cluster beyond its capacity and scale it to accomodate that
* Upgrade the cluster with a newer Kubernetes version
* Destroy the cluster

## Where

* GCP
* AWS
* Azure
* DigitalOcean

## Checklist

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

## TODO

### GKE

- [X] Code
- [ ] Blog
- [ ] Video

### EKS

- [X] Code
- [ ] Blog
- [ ] Video

### AKS

- [X] Code
- [ ] Blog
- [ ] Video

### DOKS

- [X] Code
- [ ] Blog
- [ ] Video

### General

- [ ] Intro podcast
- [ ] Summary video
- [ ] Summary podcast

### Manage clusters with jx

TODO: Figure out what would make sense to do with `jx`
