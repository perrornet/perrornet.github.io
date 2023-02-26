---
title: k8s PVC自动扩容实践
date: 2023-02-18 17:54:22
top: true
tags:
    - devops
    - k8s
---
## 依赖
## 依赖

此实践依赖于：

1. k8s版本 >= 1.24
2. [pvc-autoresizer](https://github.com/topolvm/pvc-autoresizer) 版本 >= 0.5.0

## 安装 pvc-autoresizer

1. 克隆[pvc-autoresizer](https://github.com/topolvm/pvc-autoresizer)仓库并检出版本 0.5.0，然后构建并推送Docker镜像：

```
git clone <https://github.com/topolvm/pvc-autoresizer> && git checkout v0.5.0 && cd pvc-autoresizer && docker build -t pvc-autoresizer:0.5.0 . && docker push pvc-autoresizer:0.5.0

```

1. 添加pvc-autoresizer的Helm repo:

```
helm repo add pvc-autoresizer <https://topolvm.github.io/pvc-autoresizer/>

```

1. 准备`values.yaml`文件，内容如下：

```
# config from <https://github.com/topolvm/pvc-autoresizer/blob/main/charts/pvc-autoresizer/values.yaml>
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

1. 安装pvc-autoresizer:

```
helm install --create-namespace --namespace pvc-autoresizer pvc-autoresizer pvc-autoresizer/pvc-autoresizer --values ./values.yaml

```

1. 检查是否安装成功:

```
kubectl get pod -n pvc-autoresizer | grep pvc-autoresizer

```

## 创建StatefulSet以及存储类

1. 编写`stateful-set.yaml`文件，内容如下:

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

1. 部署StatefulSet和存储类:

```
kubectl apply -f ./stateful-set.yaml

```

## 测试

1. 进入pod并查看挂载目录大小，可以看到`/data`目录只使用了1%的空间

```
df -h

```

2. 测试自动扩容，写入文件:

```
dd if=/dev/zero of=1G.file bs=50M count=20

```

3. 检查pvc-autoresizer pod 日志:

   ![/medias/1666863168702.jpg](/medias/1666863168702.jpg)

4. 再次检查pod挂载目录大小, 可以看见挂载目录已经使用了100%:`/dev/sdf 975.9M 959.9M 0 100% /data`
5. 等待一段时间后，再次查看挂载目录大小，可以看见已经扩容成功:`/dev/sdf 1.9G 960.4M 1007.4M 49% /data`😋😋😋😋😋😋

![/medias/1666863439959.jpg](/medias/1666863439959.jpg)
