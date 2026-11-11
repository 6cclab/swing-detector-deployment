# swing-detector-deployment

Kustomize manifests that ArgoCD syncs for swing-detector (api, worker, admin images under `ghcr.io/6cclab/swing-detector-*`). Namespace `swing-detector`. The app repo is not checked out under ~/projects, and there is no README.

- Layout: `base/` (api x2, worker x4, admin x1, Redis, pgvector Postgres 16, migrate Job, HomelabSecret, `ollama-text` at replicas 0), `envs/dev/` (ingress, body-limit middleware, Postgres TCP route, image tags).
- Commands: `kustomize build envs/dev` (standard Kustomize; not in a repo file).
- Image tags: three `images:` entries in `envs/dev/kustomization.yaml`. Git history shows CI commits `deploy api|worker|admin to <sha>`, so do not hand-edit. The migrate Job uses the api image.
- Sync waves: Redis/Postgres 0, migrate Job 1, Deployments 2.
- Hosts: `swing.apps.dev-01.6cclab.dev` (api), `swing-admin.apps.dev-01.6cclab.dev` (admin), `swing-db.dev-01.6cclab.dev` (Postgres via Traefik HostSNI).
- Secrets: `base/infisical.yaml` is a `HomelabSecret`, Infisical project `d5b5e6bf-5c59-4ea3-ae8f-7d72ac65438d`, producing `swing-detector-secrets`.
- Ollama: worker uses the shared `ollama-external.ollama.svc:11434`, model `qwen3.8:latest` for text and vision (`base/deployment.yaml:121-128`). Change the model in both env pairs.
- S3 is Garage at `garage.garage.svc:3900`.

## Done means
- `kustomize build envs/dev` renders cleanly and the diff touches only what you meant to change.
- Merging to `main` does not deploy: ArgoCD app `swing-detector` has no automated sync
  (homelab `services/.../application.yaml`). Run `argocd app diff swing-detector`, then sync it by hand.
