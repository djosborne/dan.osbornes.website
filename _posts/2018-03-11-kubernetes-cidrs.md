---
title: Understanding the three Kubernetes CIDRs
categories: kubernetes
tags: calico cidr
---

There are three key CIDRs to be aware of in Kubernetes:

1. Node cidr (sometimes called host cidr)
1. Pod cidr (sometimes called cluster cidr)
1. Service cidr

This post will go into some more detail on those three address ranges, but here are the key takeaways:

- None of these ranges can overlap.
- Each component may need to be individually configured with these addresses
- Each component must agree on the values.
- Calico and Kubernetes must agree on the Pod cidr.

### Node cidr

The address range of your nodes. Even though you don't explicitly configure Kubernetes with this address, it
is mentioned in this post as it is imperitive that your Node cidr does not conflict with the other two cidrs.

### Pod cidr

The address range of your Pods. Several components will need to be told which address range you are
using for your pods so that they can identify pod traffic:

- `kubelet --pod-cidr`
- `kube-proxy --cluster-cidr`
- `kube-controller-manager --cluster-cidr`

### Service cidr

The address range of your Kubernetes services. When you create a service, its cluster IP will be allocated from this
range. This address is configured on the kube-apiserver:

- `kube-apiserver --service-cluster-ip-range`

## Finding out what cidrs are

Unfortunately, Kubernetes offers no easy / standardized way of checking what it was configured with for these ranges.
You'll instead need to check what flags and defaults were used manually on each component.

## Calico's interaction with the address ranges

For Calico networking to function correctly, **Calico and Kubernetes must agree on the Pod cidr**. This means that
all your [Calico pools][pool-reference] must lie within the configured pod cidr.

## Resources

- [https://kubernetes.io/docs/getting-started-guides/scratch/](https://kubernetes.io/docs/getting-started-guides/scratch/)
- [https://kubernetes.io/docs/reference/generated/kube-apiserver/](https://kubernetes.io/docs/reference/generated/kube-apiserver/)
- [https://kubernetes.io/docs/reference/generated/kube-controller-manager/](https://kubernetes.io/docs/reference/generated/kube-controller-manager/)
- [https://kubernetes.io/docs/reference/generated/kubelet/](https://kubernetes.io/docs/reference/generated/kubelet/)

[pool-reference]: https://docs.projectcalico.org/latest/reference/calicoctl/resources/ippool
