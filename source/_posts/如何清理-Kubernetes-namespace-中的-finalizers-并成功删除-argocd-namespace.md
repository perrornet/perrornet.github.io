---
title: 如何清理 Kubernetes namespace 中的 finalizers 并成功删除 argocd namespace
date: 2023-02-18 18:03:14
tags:
    - devops
    - k8s
---
在删除 Kubernetes 集群中的资源时，我遵循以下步骤：

1. 删除所有 `deploy`。
2. 删除所有 `configmap`。
3. 删除所有 `namespace`。

&nbsp;&nbsp;然而，我发现在删除 `namespace` 时出现了问题。尝试了官方提供的命令
`kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`
但是 `namespace` 删除仍然被卡住了。我注意到该 `namespace` 下存在多个 `argoproj.io/v1alpha1/applications` 资源，并且所有资源都包含一个 `finalizers` 字段。通过搜索引擎，我了解到只有删除所有资源 YAML 文件中的 `finalizers` 字段才能解决 `namespace` 删除被卡住的问题。因此，我直接删除了 `finalizers` 字段并更新了所有资源，直到所有资源都被修改好为止。最终，`namespace` 能够被成功删除。
&nbsp;&nbsp;在 Kubernetes 中，删除某些资源时，这些资源可能具有关联的其他资源，这些关联资源需要被清理，以便能够顺利删除要删除的资源。当某个 `namespace` 中存在关联资源时，删除该 `namespace` 可能会失败。在这种情况下，Kubernetes 会在 `namespace` 对应的 `finalizers` 列表中添加一些条目，以确保所有关联资源被清理，然后再删除 `namespace`。在删除 `namespace` 时，Kubernetes 会检查 `finalizers` 列表中的所有条目，确保这些条目对应的所有资源已被清理，然后才会继续删除 `namespace`。
