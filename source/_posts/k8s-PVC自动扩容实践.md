---
title: k8s PVC Auto Scaling Practice
date: 2023-02-18 17:54:22
top: true
tags:
    - devops
    - k8s
---
## Dependencies
This practice depends on:

1. k8s version >= 1.24
2. [pvc-autoresizer](https://github.com/topolvm/pvc-autoresizer) version >= 0.5.0

## Install pvc-autoresizer

1. Clone the [pvc-autoresizer](https://github.com/topolvm/pvc-autoresizer) repository and checkout version 0.5.0, then build and push the Docker image:

```
git clone <https://github.com/topolvm/pvc-autoresizer> && git checkout v0.5.0 && cd pvc-autoresizer && docker build -t pvc-autoresizer:0.5.0 . && docker push pvc-autoresizer:0.5.0

```

1. Add the pvc-autoresizer Helm repo:

```
helm repo add pvc-autoresizer <https://topolvm.github.io/pvc-autoresizer/>

```

1. Prepare the `values.yaml` file with the following content:

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

1. Install pvc-autoresizer:

```
helm install --create-namespace --namespace pvc-autoresizer pvc-autoresizer pvc-autoresizer/pvc-autoresizer --values ./values.yaml

```

1. Check if the installation was successful:

```
kubectl get pod -n pvc-autoresizer | grep pvc-autoresizer

```

## Create StatefulSet and Storage Class

1. Write the `stateful-set.yaml` file with the following content:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: test-pvc-autoresizer
  namespace: staging
  annotations:
    resize.topolvm.io/enabled: "true" # Must be present to auto-scale
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
        annotations:  # Must be present to auto-scale
            resize.topolvm.io/storage_limit: 8Gi # Maximum scaling size
            resize.topolvm.io/threshold: 20%
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "test-pvc-autoresizer"
        resources:
          requests:
            storage: 1Gi

```

1. Deploy StatefulSet and Storage Class:

```
kubectl apply -f ./stateful-set.yaml

```

## Test

1. Enter the pod and check the mounted directory size. You can see that the `/data` directory is only using 1% of its space:

```
df -h

```

2. Test automatic scaling by writing a file:

```
dd if=/dev/zero of=1G.file bs=50M count=20

```

3. Check the pvc-autoresizer pod log:

[![ppCS1Zn.jpg](https://s1.ax1x.com/2023/02/27/ppCS1Zn.jpg)](https://imgse.com/i/ppCS1Zn)

4. Check the mounted directory size again. You can see that the mounted directory has already reached 100% usage: `/dev/sdf 975.9M 959.9M 0 100% /data`
5. After waiting for some time, check the mounted directory size again. You can see that it has successfully scaled: `/dev/sdf 1.9G 960.4M 1007.4M 49% /data`😋😋😋😋😋😋
[![ppCSQqs.jpg](https://s1.ax1x.com/2023/02/27/ppCSQqs.jpg)](https://imgse.com/i/ppCSQqs)
