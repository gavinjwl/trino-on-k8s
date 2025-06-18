# Trino on K8S

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
