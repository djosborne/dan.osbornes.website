---
title: Calico with Helm
categories: calico taint toleration
---

Helm is a great tool for packaging kubernetes applications. It would make installing Calico
a lot easier by exposing various settings through a simplified configuration api. However:

- Helm requires tiller (its server-side pod) to be running
- Tiller needs a `Ready` node to run on
- **A ready node needs its networking plugin installed** before it can be Ready.

So tiller needing Calico, and Calico needing tiller was a classic catch-22.

Until now! Thanks to
"[Taint Nodes By Condition](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/#taint-nodes-by-condition)".
This moves the `reason:NetworkPluginNotReady` into a node's taints, so that pods can choose to run even though the node
isn't ready. Previously, this was only possible by bypassing the default scheduler. But now any pod can do it. Including Tiller!

Lets try it out, via the following process:

1. Create a cluster with `TaintNodesByCondition` and `TaintBasedEvictions` enabled.
2. Install Tiller.
3. Install Calico.

### Create a Cluster

To create a compatible cluster, we'll need to flip on `TaintNodesByCondition` and `TaintBasedEvictions`. Since these features are not GA, they sit behind feature flags. So we'll set the flags on the following components:

- controller-manager --feature-gates TaintNodesByCondition=true,TaintBasedEvictions=true
- api-server --feature-gates TaintNodesByCondition=true
- scheduler --feature-gates TaintNodesByCondition=true

We'll do this with


### Other Tidbits

**Kubeadm config**

```
controllerManagerExtraArgs:
  feature-gates: TaintNodesByCondition=true,
apiServerExtraArgs:
  feature-gates: TaintNodesByCondition=true
schedulerExtraArgs:
  feature-gates: TaintNodesByCondition=true
```
