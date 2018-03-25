---
title: Calico with Minikube
categories: calico kubernetes minikube
---

Minikube is as great way to try out Kubernetes on your local machine in an isolated environment. Its also the perfect
way to try out Calico, but can be difficult to set up initially because **Calico's default IP pool overlaps with the node cidr, and is different than the pod cidr that Minikube uses by default.** This violates the [core requirements of the kubernetes cidr ranges]({% post_url 2018-03-11-kubernetes-cidrs %}.

There's a few ways to untangle this, but for now, the easiest way is to **change the range Calico uses to match the range Minikube uses.**

### Launching Calico with Minikube's pod CIDR

1. Start Minikube with the following flags:

   ```
   minikube start --network-plugin=cni \
       --extra-config=kubelet.PodCIDR=10.0.0.0/16 \
       --extra-config=proxy.ClusterCIDR=10.0.0.0/16 \
       --extra-config=controller-manager.ClusterCIDR=10.0.0.0/16
   ```


   >Note: You'll notice that your Node will become Ready even though you haven't installed Calico yet. That is OK -
   I'll explain why this occurs at the end of this post.

2. Download a Calico self-hosted manifest. Do not `kubectl apply` it directly, as we must modify one field first.
   You can use any of
   [the calico yamls from the official site][calico-yamls].

   I'm going to use the standard hosted manifest:

       curl -Lo calico.yaml https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/calico.yaml


3. Change the value of `CALICO_IPV4POOL_CIDR` from `192.168.0.0/16` to `10.0.0.0/16` in the manifest:

       sed -i 's?192.168.0.0/16?10.0.0.0/16?g' calico.yaml

4. Apply it:

       kubectl apply -f calico.yaml

Wait until Calico is running, then you may start running your workloads. If you start pods before Calico has finished installing, those pods will be launched on the briged network, so networkpolicy will not function properly, and you may encounter IP conflicts.

### Demo Policy Validation

I recomend following Calico's [simple policy demo][policy-demo] on this cluster to confirm that policy and connectivity is working as expected.

### Warning: Do not launch pods before Calico is Running

You may notice that your Kubernetes node will enter a ready state and be able to run before Calico is even installed.
This is because Minikube sets up a basic CNI configuration for you:

```
minikube ssh cat /etc/cni/net.d/k8s.conf
{
  "name": "rkt.kubernetes.io",
  "type": "bridge",
  "bridge": "mybridge",
  "mtu": 1460,
  "addIf": "true",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "subnet": "10.1.0.0/16",
    "gateway": "10.1.0.1",
    "routes": [
      {
        "dst": "0.0.0.0/0"
      }
    ]
  }
}
```

This network configuration does not support cross-node communication, but that is OK for minikube since it is a single-node
cluster creation tool.

Calico's installation procedure will place a lexigraphically higher named CNI configuration file on the node, so that
it will be used instead of the above default configuration. If a pod is run with this network before Calico is installed,
Calico will not recognize the traffic as coming from a pod, and will block it.

[policy-demo]: https://docs.projectcalico.org/latest/getting-started/kubernetes/tutorials/simple-policy
[calico-yamls]: https://docs.projectcalico.org/latest/getting-started/kubernetes/installation/hosted
