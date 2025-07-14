# Getting Started

## Considerations and Limitations

- OSS Trino doesn't support AWS Lakeformation ([re:Post](https://repost.aws/questions/QUSrcH4oLjRZ-U3xCGNmqXrg/can-you-use-oss-trino-on-emr-with-lake-formation-access-controls)), but [StarBurst does](https://docs.starburst.io/latest/security/aws-lake-formation.html).

## 預先部署資料庫

預先部署好 PostgreSQL 或 MySQL 並執行資料表建立；使用 `runMigrationsEnabled: false` 關閉資料庫的初始化避免啟動延遲。初始化 SQL 腳本參考: [連結](https://github.com/trinodb/trino-gateway/blob/main/gateway-ha/src/migrations/gateway-ha.sql)
  
```yaml
# Should be able to leverage ConfigMap or Secrets in K8S
# Check out: https://trinodb.github.io/trino-gateway/installation/#helm
dataStore:
  jdbcUrl: jdbc:postgresql://postgres:5432/trino_gateway_db
  user: USER
  password: PASSWORD
  driver: org.postgresql.Driver
  queryHistoryHoursRetention: 24
  runMigrationsEnabled: false
```

## AWS EKS

```bash
eksctl create iamserviceaccount -f eksctl-config.yaml --approve
eksctl create podidentityassociation -f eksctl-config.yaml --approve
eksctl create nodegroup  -f eksctl-config.yaml
```

## Helm charts

### Trino cluster

```bash
helm upgrade trino-0 trino/trino --namespace trino --values helm-trino/trino-values.yaml --install
```

### Trino Gateway

```bash
kubectl create cm routing-rules --namespace trino --from-file helm-trino-gateway/routing-rules.yaml
helm upgrade trino-gateway trino/trino-gateway --namespace trino --values helm-trino-gateway/trino-gateway-values.yaml --install
```

### Configuration

- Configure Trino cluster behind Trino Gateway

  ```properties
  # config.properties
  http-server.process-forwarded=true
  ```

- Configure Trino Gateway behind load balencer or Ingress

  Trino gateway

  ```yaml
  # Helm chart values
  serverConfig:
    http-server.process-forwarded: true
  ```

- 若是回傳的資料較大則需要調整以下參數

  ```yaml
  proxyResponseConfiguration:
    responseSize: 50MB
  ```

## Health check

- Healthing check with two endpoints: `/trino-gateway/livez` and `/trino-gateway/readyz`. [reference](https://trinodb.github.io/trino-gateway/operation/#trino-gateway-health-endpoints)

## Trouble shooting

```bash
SCHEMA=""
SAMPLE_STATEMENT=""
trino --catalog iceberg --schema $SCHEMA --debug --execute $SAMPLE_STATEMENT

trino --catalog iceberg --schema default --execute 'select 1' --debug localhost:8080
trino --catalog iceberg --schema default --execute 'select 10' --source airflow --debug localhost:8080
```
