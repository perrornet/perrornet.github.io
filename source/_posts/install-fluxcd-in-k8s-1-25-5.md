---
title: install fluxcd in k8s 1.25.5
date: 2023-02-18 18:04:41
tags:
   - devops
   - k8s 
---
1. install the flux cli: `brew install fluxcd/tap/flux`
2. show the flux cli version: `flux version`
```
   ➜  ~ flux version
flux: v0.39.0
helm-controller: v0.26.0
image-automation-controller: v0.26.1
image-reflector-controller: v0.22.1
kustomize-controller: v0.30.0
notification-controller: v0.28.0
source-controller: v0.31.0
```
3. create git repo on github
4. set GITHUB_TOKEN env
5. install the flux cd to k8s: `flux bootstrap github --owner=<you-github-name> --repository=<you-repo> --path=./ --components-extra=image-reflector-controller,image-automation-controller  --read-write-key --branch=main`, *Don't forget `--components-extra=image-reflector-controller,image-automation-controller`*, Only when this parameter is used can image-related pods be created.
6. wait flux cd pod up.
   