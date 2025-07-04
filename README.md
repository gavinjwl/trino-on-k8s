# Trino on K8S

```mermaid
graph LR
    client["client"]
    client --> ingress
    subgraph "K8S Cluster"
        ingress --> trino_gateway["Trino Gateway"]
        subgraph trino_gateway["Trino Gateway"]
            routing_rules@{ shape: diamond, label: "Routing
            Rules" }
        end
        routing_rules --> trino_cluster1["Trino Cluster"] & trino_cluster2["Trino Cluster"] & trino_cluster3["Trino Cluster"]
    end
    trino_gateway <--->|States| postgres@{ shape: database, label: "PostgreSQL" }
    trino_cluster1["Trino Cluster"] & trino_cluster2["Trino Cluster"] & trino_cluster3["Trino Cluster"] <--> datalake@{ shape: lin-cyl, label: "Data Lake" }
```

## Considerations and Limitations

- OSS Trino doesn't support AWS Lakeformation ([re:Post](https://repost.aws/questions/QUSrcH4oLjRZ-U3xCGNmqXrg/can-you-use-oss-trino-on-emr-with-lake-formation-access-controls)), but [StarBurst does](https://docs.starburst.io/latest/security/aws-lake-formation.html).

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

### Trouble shooting

```bash
SCHEMA=""
SAMPLE_STATEMENT=""
trino --catalog iceberg --schema $SCHEMA --debug --execute $SAMPLE_STATEMENT
```
