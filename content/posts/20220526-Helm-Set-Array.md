---
draft: false
title: Helm --set, and arrays
description: a quick reference for image pull secrets and k8s service accounts

date: 2022-05-27T19:00:00+01:00
tags: 
    - kubernetes
    - helm
    - terraform
categories:
    - helm
    - references
    - terraform
---
Had a bit of nightmare today trying to get `helm install <name> <chart> --set` to work when it wanted a list. Here's how I got past it. The example uses [external-dns](https://github.com/kubernetes-sigs/external-dns) but could apply to pretty much any chart
## Solution

Here's the command I was running, but kept getting the error `at <.Values.domainFilters>: range can't iterate over domain.example.com`

```
helm install external-dns external-dns/external-dns \
-n external-dns --values ./files/external-dns/values.yaml \
--set logLevel=debug \
--set domainFilters="domain.example.com"  \
--post-renderer ./kustomize/kustomize.sh --dry-run
```

It turns out (thanks to [this](https://github.com/helm/helm/issues/1987) issue!) that you need to wrap the values up in `{ }`

```bash
helm install external-dns external-dns/external-dns \
-n external-dns --values ./files/external-dns/values.yaml \
--set logLevel=debug \
--set domainFilters="{domain.example.com}"  \
--post-renderer ./kustomize/kustomize.sh --dry-run
```

This also applies to terraform if you go down that route. Note we need to do extra `{ }` if doing a string interpolation:

```terraform
resource "helm_release" "external-dns" {
  name       = "external-dns"
  repository = "https://charts.bitnami.com/bitnami"
  chart      = "redis"
  version    = "6.0.1"

  values = [
    "${file("values.yaml")}"
  ]

  set {
    name  = "domainFilters"
    value = "{${var.dns-zone}}"
    type  = "string"
  }
}
```

There's also some good reading in the references, that gives some other methods for maps etc.
## References
[Helm Issue 1987](https://github.com/helm/helm/issues/1987)

[Helm Docs - Limitations of --set](https://github.com/helm/helm/blob/7cbed3f77bd771dfb46c63733c497e8c05225422/docs/using_helm.md#the-format-and-limitations-of---set)



