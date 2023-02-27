---
title: install fluxcd in k8s 1.25.5
date: 2023-02-18 18:04:41
tags:
   - devops
   - k8s 
---
Here are the steps to use Flux CD for GitOps on Kubernetes:

1. Install Flux CLI: Use the command `brew install fluxcd/tap/flux` to install.
2. Display Flux CLI version: Use the command `flux version` to display Flux CLI version information.
3. Create a Git repository on GitHub.
4. Set the GITHUB_TOKEN environment variable.
5. Use the command `flux bootstrap github` to install Flux CD into the Kubernetes cluster. In the command, specify the GitHub username and repository name, read/write key, branch, and the components to use (including image-reflector-controller and image-automation-controller).
6. Wait for the Flux CD Pod to start and complete initialization.

Note that only with the `--components-extra=image-reflector-controller,image-automation-controller` parameter can you create pods related to images.
