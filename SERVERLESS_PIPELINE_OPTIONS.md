layout: page 
title: "About Me, @mallory-jpg" 
permalink: /about-me

# Serverless Pipeline Options: A Quick Comparison of EC2, ECS, & Kubernetes
by `@mallory-jpg`

### Vocab
**Serverless**: no servers, has managed infrastrucure

**Silo Effect**: resources are not shared between units

**Virtual Machine (VM)**: 

## EC2 vs ECS vs Kubernetes
### EC2
* Not managed by AWS, must manage pipelines manually
* VMs run 24/7 regardless of whether they're doing work (like running Docker containers)
* Can't scale instances up and down easily: must size EC2 to accomodate largest workload
* Used alongside Compute Engine

### ECS
* Totally managed infrastructure: auto-scaling, auto-provisioning
* Only charged when working (like a Docker container is running)

### Kubernetes (on AWS)
* Managed infrastructure
* Must size nodes manually
* Node pools are manually sized per cluster: edge node executor is small and runs 24/7
* Expensive: must size EC2 executor to size of max workload
* No *silo effect* so if one node goes bad, all are affected due to shared resources
