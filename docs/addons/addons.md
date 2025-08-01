---
title: Sveltos - Kubernetes Add-on Controller | Manage Kubernetes Add-ons with Ease
description: Sveltos is an application designed to manage hundreds of clusters by providing declarative APIs to deploy Kubernetes add-ons across multiple clusters.
tags:
    - Kubernetes
    - add-ons
    - helm
    - kustomize
    - clusterapi
    - multi-tenancy
    - Sveltos
authors:
    - Gianluca Mardente
---

## What is Sveltos?

[Sveltos](https://github.com/projectsveltos "Manage Kubernetes add-ons") is a set of Kubernetes controllers that run in the management cluster. From the management cluster, Sveltos can manage add-ons and applications on a fleet of managed Kubernetes clusters.

Sveltos comes with support to automatically discover [ClusterAPI](https://github.com/kubernetes-sigs/cluster-api) powered clusters, but it doesn't stop there. You can easily [register](../register/register-cluster.md) any other cluster (on-prem, Cloud) and manage Kubernetes add-ons seamlessly.

![Sveltos managing clusters](../assets/multi-clusters.png)

## How it works?

[ClusterProfile](https://github.com/projectsveltos/sveltos-manager/blob/main/api/v1beta1/clusterprofile_types.go "ClusterProfile to manage Kubernetes add-ons") and [Profile](https://github.com/projectsveltos/sveltos-manager/blob/main/api/v1beta1/profile_types.go "Profile to manage Kubernetes add-ons") are the CustomerResourceDefinitions used to instruct Sveltos which add-ons to deploy on a set of clusters.

- __ClusterProfile__: It is a cluster-wide resource. It can match any cluster and reference any resource regardless of their namespace.

- __Profile__: It is a namespace-scoped resource that is specific to a single namespace. It can only match clusters and reference resources within its own namespace.

By creating a **ClusterProfile**, **Profile** instances, we can easily deploy the following points across a set of Kubernetes clusters.

- Helm charts
- Resources assembled with Kustomize
- Raw Kubernetes YAML/JSON manifests

Define which Kubernetes add-ons to deploy and where:

1. Select one or more clusters using a Kubernetes [label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors "Kubernetes label selector");
1. List the Kubernetes add-ons that need to be deployed on the selected clusters

Simple as that!

## Example: Deploy Kyverno using a ClusterProfile

The below example deploys a Kyverno Helm chart in every cluster with the label selector set to `env=prod`.

### Step 1: Register Clusters with Sveltos

The first step is to register clusters with Sveltos to receive the required addons and deployments. If the clusters are not registered yet, follow the instructions outlined [here](../register/register-cluster.md). This applies to non-CAPI clusters.


```bash
$ kubectl get sveltosclusters -n projectsveltos --show-labels

NAME        READY   VERSION          LABELS
cluster12   true    v1.26.9+rke2r1   sveltos-agent=present
cluster13   true    v1.26.9+rke2r1   sveltos-agent=present
```

!!! note
    The CAPI clusters are registered in the **projectsveltos** namespace. If you register the clusters in a different namespace, update the command above.

### Step 2: Create the ClusterProfile

The second step is to create a `ClusterProfile` and apply it to the **management** cluster.

!!! example "Sveltos ClusterProfile Kyverno"
    ```yaml
    cat > clusterprofile_kyverno.yaml <<EOF
    apiVersion: config.projectsveltos.io/v1beta1
    kind: ClusterProfile
    metadata:
    name: kyverno
    spec:
    clusterSelector:
        matchLabels:
        env: prod
    syncMode: Continuous
    helmCharts:
    - repositoryURL:    https://kyverno.github.io/kyverno/
        repositoryName:   kyverno
        chartName:        kyverno/kyverno
        chartVersion:     v3.3.3
        releaseName:      kyverno-latest
        releaseNamespace: kyverno
        helmChartAction:  Install
    EOF
    ```

```bash
$ kubectl apply -f "clusterprofile_kyverno.yaml"

$ sveltosctl show addons

+--------------------------+---------------+-----------+----------------+---------+-------------------------------+------------------+
|         CLUSTER          | RESOURCE TYPE | NAMESPACE |      NAME      | VERSION |             TIME              | CLUSTER PROFILES |
+--------------------------+---------------+-----------+----------------+---------+-------------------------------+------------------+
| projectsveltos/cluster12 | helm chart    | kyverno   | kyverno-latest | 3.2.5   | 2023-12-16 00:14:17 -0800 PST | kyverno          |
| projectsveltos/cluster13 | helm chart    | kyverno   | kyverno-latest | 3.2.5   | 2023-12-16 00:14:17 -0800 PST | kyverno          |
+--------------------------+---------------+-----------+----------------+---------+-------------------------------+------------------+
```

![Sveltos in action](../assets/addons.png)

![Sveltos in action](../assets/addons_deployment.gif)

!!! note
    If you are not aware of the `sveltosctl` utility, have a look at the installation documentation found [here](../getting_started/sveltosctl/sveltosctl.md).

## More Resources

The easiest way to try out Sveltos is by following the [Quick Start](../getting_started/install/quick_start.md) guide. If you prefer a video demonstration, watch the [Sveltos introduction](https://www.youtube.com/watch?v=Ai5Mr9haWKM "Sveltos introduction: Kubernetes add-ons management") video on YouTube.
