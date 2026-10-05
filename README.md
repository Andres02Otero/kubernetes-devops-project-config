# DevOps with Kubernetes - Project configuration

Kubernetes configuration of the course project (todo app), deployed with ArgoCD. The application code and the CI workflows live in a separate repository: [kubernetes-devops-submissions](https://github.com/Andres02Otero/kubernetes-devops-submissions) (exercise 4.10: different repositories for code and configuration).

## Layout

```
base/                 manifests shared by both environments
overlays/staging/     namespace project-staging
overlays/production/  namespace project
argocd/               ArgoCD Applications (applied once with kubectl)
```

- **staging** follows `main` of this repository. Every commit to `main` of the code repository that changes application code builds new images, and its workflow commits the new image tags to `overlays/staging/kustomization.yaml`. The broadcaster only logs the messages and the database is not backed up.
- **production** changes only when a tag is pushed to the code repository. Its workflow builds the images with the tag name, sets them in `overlays/production/kustomization.yaml` and pins the base to the current commit of this repository (`//base?ref=<sha>`), so later changes in `base/` reach staging but not production until the next release. Production adds the daily database backup to Google Cloud Storage and the external service the broadcaster forwards to (a generic echo server).

## Setup

```bash
kubectl apply -n argocd -f argocd/production.yaml
kubectl apply -n argocd -f argocd/staging.yaml
```

Secrets applied outside ArgoCD: `storage-sa-key` (service account key for the backups, namespace `project`). NATS runs in the cluster as a Helm release (`my-nats`, namespace `nats`).
