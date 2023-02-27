---
title: helm error: another operation (install/upgrade/rollback) is in progress, solution
date: 2023-02-18 18:01:54
top: true
tags:
    - devops
    - k8s
---
This article describes how to manage Helm Releases using Flux CD.

### Problem description

When using Helm Release, sometimes an error message appears: "another operation (install/upgrade/rollback) is in progress".

### Solution

1. Run the command `helm history <name> --namespace <ns>` to view the history of the Release.
![/medias/1675238086770.jpeg](/medias/1675238086770.jpeg)
2. Run the command `flux suspend hr <name> -n <ns>` to pause the automatic upgrade of the Release.
3. Select the version with `status=pending-upgrade`, and run the command `helm uninstall <name> --namespace <ns> <REVISION>` to uninstall that version.
4. Run the command `flux resume helmrelease <name> -n <ns>` to restart the Release.

The above steps can solve this error.
