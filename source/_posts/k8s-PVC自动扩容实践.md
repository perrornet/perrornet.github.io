---
title: k8s PVC自动扩容实践
date: 2023-02-18 17:54:22
top: true
tags:
    - devops
    - k8s
---


依赖:
1. k8s >= 1.24
2. [pvc-autoresizer](https://github.com/topolvm/pvc-autoresizer) >= 0.5.0

一、安装 pvc-autoresizer
* build  and push docker 镜像 `git clone https://github.com/topolvm/pvc-autoresizer && git checkout v0.5.0 && cd pvc-autoresizer && docker build -t pvc-autoresizer:0.5.0 . && docker push pvc-autoresizer:0.5.0`
* 添加 helm repo: `helm repo add pvc-autoresizer https://topolvm.github.io/pvc-autoresizer/`
  <!-- more -->


* values.yaml
```
# config from https://github.com/topolvm/pvc-autoresizer/blob/main/charts/pvc-autoresizer/values.yaml
image:
  # image.repository -- pvc-autoresizer image repository to use.
  repository: perrorone/pvc-autoresizer

  # image.tag -- pvc-autoresizer image tag to use.
  # @default -- `{{ .Chart.AppVersion }}`
  tag:  v0.5.0

controller:
  # controller.replicas -- Specify the number of replicas of the controller Pod.
  replicas: 1

  args:
    # controller.args.prometheusURL -- Specify Prometheus URL to query volume stats.
    # Used as "--prometheus-url" option
    prometheusURL: <you_prometheus_url>
```

* 安装 pvc-autoresizer: `helm install --create-namespace --namespace pvc-autoresizer pvc-autoresizer pvc-autoresizer/pvc-autoresizer --values ./values.yaml"`
* 检查是否成功 `kubectl get pod -n pvc-autoresizer | grep pvc-autoresizer`

二、 创建StatefulSet以及存储类
* 编写stateful-set.yaml:
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: test-pvc-autoresizer
  namespace: staging
  annotations:
    resize.topolvm.io/enabled: "true" # 必须存在, 才能自动扩容
parameters:
  type: pd-ssd
provisioner: pd.csi.storage.gke.io
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: test-pvc-autoresizer
  namespace: staging
spec:
  selector:
    matchLabels:
      app: test-pvc-autoresizer
  serviceName: "test-pvc-autoresizer"
  replicas: 1
  template:
    metadata:
      labels:
        app: test-pvc-autoresizer
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: test-pvc-autoresizer
          image: perrorone/go-file:v1.0.0
          ports:
            - containerPort: 80
              name: http
          livenessProbe:
            httpGet:
              path: /health
              port: http
          readinessProbe:
            httpGet:
              path: /health
              port: http
          volumeMounts:
            - name: test-pvc-data
              mountPath: /data
          resources:
            requests:
              cpu: 200m
              memory: 300Mi
            limits:
              cpu: 500m
              memory: 500Mi
  volumeClaimTemplates:
    - metadata:
        name: test-pvc-data
        annotations:  # 必须存在, 使得自动创建的 PVC携带以下设置想才能自动扩容
            resize.topolvm.io/storage_limit: 8Gi # 最大扩容大小
            resize.topolvm.io/threshold: 20%
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "test-pvc-autoresizer"
        resources:
          requests:
            storage: 1Gi
```
* 部署: `kubectl apply -f ./stateful-set.yaml`

三、测试
* 进入pod中检查挂载目录大小, `df -h` 输出可以看到`/data`目录只使用了1%空间
* 写入文件测试自动扩容: `dd if=/dev/zero of=1G.file bs=50M count=20`
* 检查pvc-autoresizer pod 日志
  ![](/medias/1666863168702.jpg)

* 再次检查pod挂载目录大小, 可以看见挂载目录已经使用了100%: `/dev/sdf                975.9M    959.9M         0 100% /data`
* 等待一会,再次查看挂载目录大小, 可以看见已经扩容成功: `/dev/sdf                  1.9G    960.4M   1007.4M  49% /data` 😋😋😋😋😋😋
  ![](/medias/1666863439959.jpg)

