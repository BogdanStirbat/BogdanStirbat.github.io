---
layout: post
title:  "Introduction to Kubernetes"
date:   2023-06-21 11:16:18 +0300
categories: jekyll update
---

### Introduction

Docker is a huge game changer in the development world. A production environment can easily be replicated in 
a development environment as well, since applications are running in containers. 

A problem that emerges is orchestrating containers. We can have a microservice application, so that we need to orchestrate multiple containers. 
A solution to this problem is [Kubernetes](https://kubernetes.io/docs/concepts/overview/), or k8s.

### Introduction to Kubernetes
As described right in the [reference documentation](https://kubernetes.io/docs/home/), Kubernetes is an open source container orchestration engine.
It is used for automating deployments and scaling of containerized applications.

Kubernetes is a complex topic. Installing Kubernetes requires specialized knowledge.
For this reason there are cloud providers offering managed Kubernetes services like Amazon EKS, Google GKE. 

On a developer machine the [minikube](https://minikube.sigs.k8s.io/docs/) project allows to quickly set up a local k8s cluster. 

### Kubernetes architecture

Kubernetes is a complex project with a complex architecture. Several of the main k8s components are the following:
- [Pods](https://kubernetes.io/docs/concepts/workloads/pods/). A Pod is the smallest deployable unit in k8s. A Pod can run one or more containers. All the containers inside a Pod share storage and network resources.
- [Nodes](https://kubernetes.io/docs/concepts/architecture/nodes/). Nodes are virtual or physical machines that host pods.
- [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/). A ReplicaSet ensures that a specific number (given as a parameter) of instances of a specific Pod are running at a specific time.
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/). Deployments  provide declarative updates for Pods and ReplicaSets.
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/). Services expose a network application that run in one or more Pods to the host operating system.

### Conclusions
Kubernetes is a complex container orchestration platform. Minikube can be used to easily get started with this topic.

### Links

https://kubernetes.io/docs/home/

https://minikube.sigs.k8s.io/docs/
