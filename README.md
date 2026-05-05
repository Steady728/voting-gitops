# voting-gitops

GitOps manifests for the Docker voting app, structured with Kustomize.

## Components

| Service  | Image                                   | Role                                          |
|----------|-----------------------------------------|-----------------------------------------------|
| `vote`   | `dockersamples/examplevotingapp_vote`   | Python web UI — accepts votes via Redis        |
| `redis`  | `redis:alpine`                          | In-memory queue for incoming votes            |
| `worker` | `dockersamples/examplevotingapp_worker` | Drains Redis and writes results to Postgres   |
| `db`     | `postgres:16`                           | Persistent vote store                         |
| `result` | `dockersamples/examplevotingapp_result` | Node.js web UI — reads live results from db   |

Traffic enters via Traefik Ingress: `vote.local` → vote UI, `result.local` → results UI.

## Repository Structure

```
base/
├── kustomization.yaml
├── namespace.yaml
├── redis.yaml
├── db.yaml
├── vote.yaml
├── result.yaml
├── worker.yaml
└── ingress.yaml
```

## Secret Management

Postgres requires a password via a Kubernetes Secret named `postgres-secret`. The secret is **not stored in this repo** — it is applied manually to the cluster out of band before deploying:

```bash
kubectl create secret generic postgres-secret \
  --namespace voting \
  --from-literal=POSTGRES_PASSWORD=<your-password>
```

The `db`, `worker`, and `result` deployments all reference this secret via `secretKeyRef`. No plaintext credentials in git.


