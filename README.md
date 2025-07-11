# Trino on K8S

## Getting Started

[Getting Started](./getting-started.md)

## What is Trino

```mermaid
flowchart LR
client --> cluster
subgraph cluster["Trino Cluster"]
direction LR
coordinate["Coordinator node"] --> work1["Worker node"]
coordinate["Coordinator node"] --> work2["Worker node"]
coordinate["Coordinator node"] --> work3["Worker node"]
end
```

## What is Trino Gateway

```mermaid
graph LR
    client["client"]
    client --> lb["Load Balancer"]

    lb["Load Balancer"] --> trino_gateway["Trino Gateway"]
    subgraph trino_gateway["Trino Gateways"]
        routing_rules@{ shape: diamond, label: "Routing
        Rules" }
    end

    routing_rules --> trino_cluster1["Trino Cluster"] & trino_cluster2["Trino Cluster"] & trino_cluster3["Trino Cluster"]
    trino_gateway <--->|States| postgres@{ shape: database, label: "PostgreSQL" }
    trino_cluster1["Trino Cluster"] & trino_cluster2["Trino Cluster"] & trino_cluster3["Trino Cluster"] <--> datalake@{ shape: lin-cyl, label: "Data Lake" }
```

## Best practices

### Trino cluster lifecycle

[reference](https://trinodb.github.io/trino-gateway/operation/#graceful-shutdown)

```mermaid
flowchart LR
1["Add Trino Cluster"] --> 2["Activate Trino Cluster"] --> 3["Serving"]
3 --> 4
4["Deactivate Trino Cluster"] --> 5["Gracful shutdown Trino cluster"] --> 6["Rmove Trino cluster"]
```

[Trino has a graceful shutdown API that can be used exclusively on workers in order to ensure that they terminate without affecting running queries, given a sufficient grace period.](https://trino.io/docs/current/admin/graceful-shutdown.html)

### [TODO] Query routing options

### Cache layer

Consider Adding Cache Layer to Reduce Requests to Trino and Improve Response Time [[Issue](https://github.com/trinodb/trino-gateway/issues/606)]

- Query identifiers: user_query_hash and generic_query_hash
- Query result cache

### Security

- TLS configuration

  ```yaml
  serverConfig:
    http-server.http.enabled: false
    http-server.https.enabled: true
    http-server.https.port: 8443
    http-server.https.keystore.path: certificate.pem
    http-server.https.keystore.key: changeme
  ```

- Authz and Authn
