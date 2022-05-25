---
draft: false
title: Image Pull Secrets and the Default Service Account
description: a quick reference for image pull secrets and k8s service accounts

date: 2022-05-25T19:00:00+01:00
tags: 
    - kubernetes
    - ghcr
categories:
    - kubernetes
    - references
---
Kubernetes is starting to bleed into my non-work time now. I find I'm mucking around with things in local clusters more and more. 

I don't particularly like making images "public" so integrating with my personal registry on `ghcr.io` was pretty important. So I'm capturing a simple, dirty way of handling it here.

> :warning: using the default account isn't generally accepted practice in production, this is ideal for development but there's better, safer ways for real workloads.

```
kubectl create secret docker-registry ghcr-pat --docker-server=https://ghcr.io  --docker-username drtbz --docker-password <pat_here>
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "ghcr-pat"}]}'
```

This is great for single repositories.

## References
[Create docker-registry secret](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line)
[ghcr docs](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)



