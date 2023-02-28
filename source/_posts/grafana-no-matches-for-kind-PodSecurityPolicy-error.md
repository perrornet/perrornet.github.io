---
title: How to fix the "no matches for kind "PodSecurityPolicy" in version "policy/v1beta1" error in Grafana
date: 2023-02-28 09:53:09
tags:
    - devops
    - k8s
---
After upgrading the cluster to Kubernetes 1.25, if Grafana was deployed previously with `rbac.spsEnabled=true` enabled, the error "grafana no matches for kind 'PodSecurityPolicy' in version 'policy/v1beta1'" will occur. This is because the previously deployed Grafana created a PodSecurityPolicy version of policy/v1beta1, which is no longer supported in Kubernetes 1.25, resulting in an error.

To resolve this issue, follow these steps:

1. Obtain Grafana's Secrets

```
> kubectl get secret -n prometheus | grep grafana
grafana                                                      Opaque               3      106d
sh.helm.release.v1.grafana.v3                                       helm.sh/release.v1   1      105d
sh.helm.release.v1.grafana.v4                                       helm.sh/release.v1   1      105d
sh.helm.release.v1.grafana.v5                                       helm.sh/release.v1   1      97d

```

2. Backup sh.helm.release.v1.grafana.v3 | sh.helm.release.v1.grafana.v4 | sh.helm.release.v1.grafana.v5

```
kubectl get secret sh.helm.release.v1.grafana.v3 -n prometheus -o yaml > release.v3.yaml
kubectl get secret sh.helm.release.v1.grafana.v4 -n prometheus -o yaml > release.v4.yaml
kubectl get secret sh.helm.release.v1.grafana.v5 -n prometheus -o yaml > release.v5.yaml

```

3. Delete sh.helm.release.v1.grafana.v3 | sh.helm.release.v1.grafana.v4 | sh.helm.release.v1.grafana.v5

```
kubectl delete secret sh.helm.release.v1.grafana.v3 -n prometheus
kubectl delete secret sh.helm.release.v1.grafana.v4 -n prometheus
kubectl delete secret sh.helm.release.v1.grafana.v5 -n prometheus

```

4. Use fluxctl reconcile grafana helmrelease

```
flux reconcile helmrelease grafana -n prometheus

```

Check Grafana helmrelease's status. If it becomes READY and no errors occur, the problem has been successfully resolved.


