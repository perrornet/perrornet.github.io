---
title: 如何在k8s中使用GCS备份ES数据
date: 2023-08-27 12:21:30
tags:
  - devops
  - k8s
---
Elasticsearch 是一个超级炫酷的开源搜索引擎，可以在各种不同的环境中使用。为了妥善保管数据，就必须得备份 ES 集群数据。鉴于公司的 k8s 集群部署在 GCP 上，自然而然想到了使用 GCS 作为备份数据存储，为了做到这一步需要以下步骤：

1. 创建一个服务帐号供 repository-gcs 插件使用，一定要注意这个服务账号必须要有gcs的权限。
2. 创建一个脚本，该脚本需要创建备份repository、检查备份任务等等。

### 配置 GCS 存储桶

首先，需要创建一个 GCS 存储桶并配置访问权限。还需要创建一个用于备份的服务账号，并赋予它存储桶的写入权限。最后，在 k8s 中配置该服务账号的密钥，以便可以使用它来访问 GCS。(由于公司集群就在GCP上部署，所以我没有这样做，而是使用了[工作负载身份联合](https://cloud.google.com/iam/docs/workload-identity-federation?hl=zh-cn))

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: gcs-credentials
type: Opaque
stringData:
  credentials.json: |
    {
      "type": "service_account",
      "project_id": "my-project",
      "private_key_id": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      "private_key": "-----BEGIN PRIVATE KEY-----\\nMIIEvQIBADANBgkqhkiG9w0B...",
      "client_email": "my-service-account@my-project.iam.gserviceaccount.com",
      "client_id": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      "auth_uri": "https://accounts.google.com/o/oauth2/auth",
      "token_uri": "https://accounts.google.com/o/oauth2/token",
      "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
      "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/my-service-account%40my-project.iam.gserviceaccount.com"
    }

```

## 配置 ES 集群

然后需要在 ES 集群中配置 GCS 备份。可以使用`repository-gcs` 插件来实现。该插件可以将 ES 数据备份到 GCS 存储桶中，并从存储桶中恢复数据。需要在每个 ES 节点上安装该插件，并在集群配置文件中添加 GCS 存储桶的详细信息。

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elastic
spec:
  nodeSets:
			...
      podTemplate:
        spec:
          serviceAccountName: es # 使用了[工作负载身份联合](https://cloud.google.com/iam/docs/workload-identity-federation?hl=zh-cn)
					...
          initContainers:
            - name: install-plugins # 安装插件
              command:
                - sh
                - -c
                - |
                  bin/elasticsearch-plugin install --batch repository-gcs
						...

```

## 创建备份计划

由于我们需要让es开启定期备份所以我们需要使用一个脚本，在ES启动后检查该任务是否存在，如果不存在则创建，为了能够将该脚本在ES中使用，我们需要为该脚本创建一个configmap。

```bash
if [ -z "$ES_URL" ]; then
  ES_URL="http://localhost:9200"
fi

if [ -z "$ES_USERNAME" ]; then
  echo "ES_USERNAME is not set"
  exit 1
fi

if [ -z "$ES_PASSWORD" ]; then
  echo "ES_PASSWORD is not set"
  exit 1
fi

if [ -z "$BUCKET" ]; then
  BUCKET="es-backup"
fi

if [ -z "$BASE_PATH" ]; then
  BASE_PATH="backup"
fi

if [ -z "$SNAPSHOT_NAME" ]; then
  SNAPSHOT_NAME="test"
fi

if [ -z "$SCHEDULE" ]; then
  SCHEDULE="0 0 14 * * ?" # 每天14点执行
fi

AUTH=$(echo -n "$ES_USERNAME:$ES_PASSWORD" | base64)
while [[ "$(curl -s -o /dev/null -w "%{http_code}" -H "Authorization: Basic $AUTH" $ES_URL)" != "200" ]]; do
    echo "Elasticsearch is not ready yet"
    sleep 1
done
echo "Elasticsearch is up and running!"

# 检查_snapshot/$SNAPSHOT_NAME是否存在，如果不存在则创建
resp=$(curl -s -o /dev/null -w "%{http_code}" -X GET  "$ES_URL/_snapshot/$SNAPSHOT_NAME" \
    -H "Accept: application/json" \
    -H "Authorization: Basic $AUTH")

if [[ $resp == "200" ]]; then
    echo "$SNAPSHOT_NAME snapshot already exists."
else
    curl -s -o /dev/null -w "%{http_code}" -X PUT "$ES_URL/_snapshot/$SNAPSHOT_NAME" \
        -H "Accept: application/json" \
        -H "content-type: application/json" \
        -H "Authorization: Basic $AUTH" \
        -d "{
            \"type\": \"gcs\",
            \"settings\": {
              \"bucket\": \"$BUCKET\",
              \"base_path\": \"$BASE_PATH\"
            }
          }" | grep -q "200" || exit 1
    echo "$SNAPSHOT_NAME snapshot has been created."
fi

# 检查snapshot policy是否存在，如果不存在则创建
resp=$(curl -s -o /dev/null -w "%{http_code}" -X GET  "$ES_URL/_slm/policy/$SNAPSHOT_NAME-snapshots" \
           -H "Accept: application/json" \
           -H "Authorization: Basic $AUTH")
if [[ $resp == "200" ]]; then
    echo "$SNAPSHOT_NAME-snapshots snapshot policy already exists."
else
    curl -s -o /dev/null -w "%{http_code}" -X PUT  "$ES_URL/_slm/policy/$SNAPSHOT_NAME-snapshots" \
        -H "Accept: application/json" \
        -H "content-type: application/json" \
        -H "Authorization: Basic $AUTH" \
        -d "{
            \"schedule\": \"$SCHEDULE\",
            \"name\": \"<$SNAPSHOT_NAME-snapshots-{now/d/H}>\",
            \"repository\": \"$SNAPSHOT_NAME\",
            \"config\": {
                \"expand_wildcards\": \"all\",
                \"ignore_unavailable\": true,
                \"include_global_state\": false
            },
            \"retention\": {
              \"expire_after\": \"30d\",
              \"min_count\": 5,
              \"max_count\": 50
            }
          }" | grep -q "200" || exit 1
    echo "$SNAPSHOT_NAME-snapshots snapshot policy has been created."
fi

echo "sleep forever"
sleep infinity
```

最终的ES配置文件为：

```bash
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elastic
spec:
  nodeSets:
    - name: default
      count: 1
      config:
        action.destructive_requires_name: true
        cluster.max_shards_per_node: 4000
      volumeClaimTemplates:
        - metadata:
            name: elasticsearch-data
          spec:
            accessModes: ["ReadWriteOnce"]
            storageClassName: premium-rwo
            resources:
              requests:
                storage: 200Gi
      podTemplate:
        spec:
          serviceAccountName: es-backup
          volumes:
            - name: create-elasticsearch-snapshot-entrypoint # 引入脚本configmap
              configMap:
                name: create-elasticsearch-snapshot-script
                defaultMode: 0744
          initContainers:
            - name: install-plugins
              command:
                - sh
                - -c
                - |
                  bin/elasticsearch-plugin install --batch repository-gcs
          containers:
            - name: elasticsearch
              resources:
                requests:
                  memory: 6Gi
                  cpu: 2000m
                limits:
                  memory: 8Gi
                  cpu: 4000m
            - name: create-elasticsearch-snapshot # 创建任务脚本配置
              image: curlimages/curl:8.2.1
              command:
                - sh
                - -c
                - sh /usr/local/bin/create_elasticsearch_snapshot.sh
              env:
                - name: ES_URL
                  value: "http://localhost:9200"
                - name: ES_USERNAME
                  value: "elastic"
                - name: ES_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: elastic-subscan-es-elastic-user
                      key: elastic
                - name: BUCKET
                  value: "es-backup"
                - name: BASE_PATH
                  value: "staging"
                - name: SNAPSHOT_NAME
                  value: "test"
                - name: SCHEDULE
                  value: "0 0 14 * * ?" # 每天14点执行
              volumeMounts:
                - name: create-elasticsearch-snapshot-entrypoint
                  mountPath: /usr/local/bin/create_elasticsearch_snapshot.sh
                  subPath: entrypoint.sh
```

在确保 ES 数据备份安全的同时，我们还可以将备份数据用于测试和开发环境。此外，我们可以在 GCS 存储桶中设置生命周期规则以自动删除旧备份，从而节省存储空间。
