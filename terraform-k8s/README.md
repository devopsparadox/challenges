# TODO

## GKE

- [X] Code
- [ ] Blog
- [ ] Video

## EKS

- [X] Code
- [ ] Blog
- [ ] Video

## AKS

- [X] Code
- [ ] Blog
- [ ] Video

## DOKS

- [X] Code
- [ ] Blog
- [ ] Video

## General

- [ ] Intro podcast
- [ ] Summary video
- [ ] Summary podcast

## Manage clusters with jx

TODO: Figure out what would make sense to do with `jx`

# Installing and Maintain Kubernetes Clusters

There are quite a few ways we could install and maintain a Kubernetes cluster. We could, for example, create nodes ourselves, install a few packages, and use `kubeadm` to install and configure the components that will consitute our cluster. However, unless you want to be in full control and you really know what you're doing, `kubeadm` is too low-level and requires too much work that can be better spent elsewhere.

We could also move to a higher level and use one of the platforms that elevate some of the pain of setting up a Kubernetes cluster with `kubeadm`. That could, for example, be [Rancher](https://rancher.com/) if you prefer open source, [OpenShift](https://www.openshift.com/) if you want to pay someone money to get an "enterprise grade cluster with support", or any one of the myriad of other options. Such options are great, but only if you are running on-prem. If you manage your own in-house datacenters, use one of those tools. They are likely a better fit for your needs than using `kubeadm`. But, if you are using one of the public clouds, they are not a good choice. They are just slightly better options than `kubeadm`.

We'll focus on public clouds. That does not mean that running a Kubernetes cluster on-prem is a bad option, but rather that public clouds have limited number of permutations we need to take into account when assessing pros and cons of each solution.

Almost all public cloud providers have a managed Kubernetes solution. We have AWS Elastic Kubernetes Service (EKS), Azure Kubernetes Service (AKS), Google Kubernetes Engine (GKE), DigitalOcean Kubernetes (DOKS), and so on and so forth. All of them offer *Managed Kubernetes* service, even though the meaning of the word "managed" varies a lot from one provider to another. Some will create, maintain, and operate the cluster for you, others will give you the service that you can use to create a fully managed Kubernetes cluster, while there are solutions where "managed Kubernetes" means, "I give you master, you figure out the rest". What is provided by which provider is something we'll explore later. For now, it is important to understand that, more often than not, "managed Kubernetes" does not mean that the provider will do all the work for you, but rather that it has one or more services we can use to create and manage Kubernetes clusters. More importantly, such a service provides integration of a Kubernetes cluster with other services. A good example is storage that is often already provided as a StorageClass and Load Balancer integration with Kubernetes Services.

Now that we clarified that we'll use managed Kubernetes clusters in one of the public cloud providers, we need to figure out how we'll create and maintain them.

We could go to cloud provider's UI and start clicking buttons, but we won't. If that's what you prefer, you're in the wrong place. While UIs might be a good way to learn basics, they cause too many problems. Clicking buttons and filling some fields with data results in unreproducible and undocumented actions. We need to define everything as code and store everything in Git. By pushing changes to, let's say, GitHub, we are creating history of changes to the state of our cluster. They can be reviewed, approved, tracked, and reproduced by anyone with sufficient permissions. Long story short, we'll discard UIs, at least for creation and maintenance of Kubernetes clusters.

An alternative to performing actions through an UI is to execute commands from a terminal window. Commands can be stored in Git and executed by both humans and, more importantly, by machines. However, not all commands are equally valid if we add idempotency into the mix of the requirements. What we need are commands that will always create the same state, no matter how many times we execute them. We need something similar to `kubectl apply` which creates resources that are missing, and updates those that were created before. As a matter of fact, we need a bit more than that. We need commands that will not only create or update resources, but also delete those that were removed from a definition. For example, if we'd use DigitalOcean and it's CLI (`doctl`) we might be tempted to execute `doctl kubernetes cluster create my-cluster`. That would create a cluster and it would work well only the first time we execute it. The command is not idempotent. If we re-run it, it will fail since the cluster already exists. We could not execute it to, let's say, upgrade Kubernetes version. So, we'd need to create a shell script that would have some kind of a conditional statement that would check whether the cluster exists. If it doesn't, it would execute `doctl kubernetes cluster create` and if it does it would run `doctl kubernetes cluster upgrade`. Even if we do that, later on we might need to update some of the cluster properties and that would require yet another conditional with yet another command (`doctl kubernetes cluster update`). On top of that, we'd need to define some variables that we could change as our needs are changing. Over time, such a script would become a nightmare to maintain. Nevertheless, having such a script would make sense, but only if we'd move back in time and go back to 1999. Today, we have configuration management tools like Chef, Puppet, Ansible, Salt, Terraform, and others. We just need to choose one of those.

The problem is that I don't have enough time to go into a lenghtly debate about pros and cons of all the major configuration management tools, yet we need to choose one. So, I'll make it a simple for you. If you already invested time and money into one configuration management tools, use it. The real issue is that I also need to choose one for the examples that follow and you will need to use what I chose if you are to follow the exercises. My choice is [Terraform](https://www.terraform.io/). It is the most widely used tool to provision and manage cloud, infrastructure, and services, especially in public clouds. It is free (open source), it has a huge community of contributors, and it is built around the idea of immutability. Many other similar tools were created long before Terraform and are designed principles of mutable infrastructure. While writing this I'm realizing that I started going down the path of justifying which Terraform is better than other tools, so I'll stop. We'll use it and it's up to you to adapt the logic we'll apply to whichever tool you prefer instead. If that's not good enough for you and you do want a deeper comparison, please send me an email, Twitter or Slack message, and I'll do my best to dive deper into that topic as a series of blog post or maybe even choose it as a subject of one of my next books.

It would be unrealistic to explore everything we might need to know to run a Kubernetes cluster, so we'll define the goals and the scope.

The object is to not to teach you everything related to managing a Kubernetes cluster. The real goal is to compare Kubernetes in a few cloud providers. By knowing pros and cons of each, you'll be able to make a more educated decision which one to use. The result of that objective is that we'll learn how to do some of the most commonly used operations. If you already made up your mind and you will stick with your hosting provider no matter what, and that is one of the vendors we'll explore, you can skip directly to the one that interests you.

We'll split what we'll do into scope, requirements, steps, and metrics.

We won't be able to explore every variation of Kubernetes. That would be close to an infinite number so we'll limit the scope to the three major public cloud providers, and add one of the smaller ones in case "big guys" are not your choice. They are as follows.

* AWS Elastic Kubernetes Service (EKS)
* Azure Kubernetes Service (AKS)
* Google Kubernetes Engine (GKE)
* DigitalOcean Kubernetes (DOKS)

The steps we'll perform are as follows.

* Create a Kubernetes cluster
* Deploy an application that uses more resources than what the cluster has allocated
* Upgrade the cluster to a newer Kubernetes version
* Destroy the cluster

We are yet to see whether we will be able to perform all those steps.

Now that we're clear about the steps, we'll define a few requirements.

* Everything should be defined as code and stored in Git
* All operations should be idempotent
* Everything should be done by Terraform

We might not be able to fully accomplish those requirements. Think of them as "if possible and practical" type of guidelines.

We need some kind of (semi) objective metrics that will help us validate different solutions.

* It is fast?
* Is it easy to install and maintain?
* Is it fault tollerant
* Is it highly-available
* Can it be scaled automatically
* Can be upgraded without downtime

Finally, to make things more interesting, I'll provide a personal opinion that you can use or disregard and stick with the more objective metrics.
