# aac-k3s

Kubernetes manifests and Helm values for the Aachen k3s cluster.

## Structure

```
manifests/          # Applied with: kubectl apply -f <file>
  apps/
    jazzam/         # JazzAM app (frontend, standards service, search service)
    rwthbigband/    # RWTH Big Band website
    dgc/            # DGC Aachen website
    default/        # Default namespace (Alex's Cloudflare tunnel)
  infrastructure/
    monitoring/     # ServiceMonitors for Prometheus (Longhorn, Traefik)
    registry/       # In-cluster container registry (port 30100)

helm/               # Applied with: helm upgrade --install
  postgres/         # Bitnami PostgreSQL values
  prometheus/       # kube-prometheus-stack values

secrets-templates/  # Redacted placeholders — copy to private repo and fill in
  jazzam/
  rwthbigband/
  default/
  postgres/
```

## Applying manifests

```bash
# Create namespace first, then apply resources
kubectl apply -f manifests/apps/jazzam/namespace.yaml
kubectl apply -f manifests/apps/jazzam/deployment.yaml
kubectl apply -f manifests/apps/jazzam/tunnel.yaml
```

## Applying Helm charts

```bash
# PostgreSQL (Bitnami)
helm upgrade --install postgres bitnami/postgresql \
  --namespace postgres --create-namespace \
  -f helm/postgres/values.yaml

# Prometheus stack
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace \
  -f helm/prometheus/values.yaml
```

## Secrets

Secrets are **not** in this repo. Use the templates in `secrets-templates/` as a
reference. The actual secrets live in the private companion repo on the self-hosted
server. Apply them before deploying the apps that reference them.

Required secrets per namespace:

| Namespace    | Secret name            | Template                                         |
|--------------|------------------------|--------------------------------------------------|
| `jazzam`     | `jazz-standards.env`   | `secrets-templates/jazzam/jazz-standards.env.yaml` |
| `jazzam`     | `tunnel-token`         | `secrets-templates/jazzam/tunnel-token.yaml`     |
| `default`    | `tunnel-token`         | `secrets-templates/default/tunnel-token.yaml`    |
| `postgres`   | `postgres-credentials` | `secrets-templates/postgres/postgres-credentials.yaml` |
| `rwthbigband`| `rwthbigband.env`      | `secrets-templates/rwthbigband/app.env.template` |
