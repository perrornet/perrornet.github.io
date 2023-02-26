---
title: install fluxcd in k8s 1.25.5
date: 2023-02-18 18:04:41
tags:
   - devops
   - k8s 
---
以下是在Kubernetes上使用Flux CD进行GitOps的步骤：

1.安装Flux CLI：使用`brew install fluxcd/tap/flux`命令进行安装。

2.显示Flux CLI版本：使用`flux version`命令显示Flux CLI版本信息。

3.在GitHub上创建Git仓库。

4.设置GITHUB_TOKEN环境变量。

5.使用`flux bootstrap github`命令将Flux CD安装到Kubernetes集群中。在命令中，需要指定GitHub用户名和仓库名、读写密钥、分支以及使用的组件（包括image-reflector-controller和image-automation-controller）。

6.等待Flux CD Pod启动并完成初始化。

请注意，只有在使用`--components-extra=image-reflector-controller,image-automation-controller`参数时，才能创建与镜像相关的Pod。
