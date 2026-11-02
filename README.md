<<<<<<< HEAD
# exam-app manifests (single source of truth)

Argo CD watches this repo. Whatever is on `main` is what runs in the cluster.
Backend and frontend each have their own self-contained folder and their own Argo CD app.

```
backend/     deployment.yaml  service.yaml  ingress.yaml  kustomization.yaml  secret.example.yaml
frontend/    deployment.yaml  service.yaml  ingress.yaml  kustomization.yaml
argocd/      backend.yaml, frontend.yaml   (Argo CD Application objects, applied once by hand)
```

- backend: port 5000, health check `/api/health`, ingress path `/api`
- frontend: nginx on port 80, ingress path `/`
- namespace `exam-app` is created by Argo CD (`CreateNamespace=true`)

## First-time setup

1. Push this repo to GitHub and replace `repoURL` in `argocd/backend.yaml` and `argocd/frontend.yaml`.
2. In each `kustomization.yaml`, replace `DOCKERHUB_USERNAME` and set `newTag` to a tag the CI
   pipeline pushed (the 7-character commit SHA).
3. Create the backend secret in the cluster (never stored in git). `DB_HOST` must be reachable from inside the cluster:
   ```
   kubectl create namespace exam-app
   kubectl -n exam-app create secret generic backend-secret \
     --from-literal=DATABASE_URL='postgres://USER:PASSWORD@DB_HOST:5432/DB_NAME'
   ```
4. Deploy the backend first, then the frontend:
   ```
   kubectl apply -f argocd/backend.yaml
   kubectl -n exam-app port-forward svc/backend 5000:5000      # curl http://localhost:5000/api/health
   kubectl apply -f argocd/frontend.yaml
   ```

## Releasing a new image

Edit `newTag` in `backend/kustomization.yaml` or `frontend/kustomization.yaml`, commit, push.
Argo CD syncs automatically.

## Notes

- Build the frontend image with `VITE_API_URL=/api` so it calls the backend through the ingress.
- Ingress class is `nginx`. On k3s (Traefik) change `ingressClassName` in both `ingress.yaml` files.
- Private registry images need an `imagePullSecret` (commented out in both deployments).
=======
# My-app-repo
>>>>>>> a10d21d329e4ab486eef662acb847eedbeabdd7b
