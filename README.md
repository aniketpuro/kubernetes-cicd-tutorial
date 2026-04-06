# Kubernetes CI/CD Tutorial (GitOps with Argo CD)

A hands-on repository that demonstrates a complete **Kubernetes CI/CD (GitOps)** workflow using **GitHub**, **GitHub Container Registry (GHCR)**, **Helm**, and **Argo CD**.

This project is maintained by **@aniketpuro**. If you fork this repo, remember to update any URLs and usernames accordingly.

---

## What You’ll Learn

- How to install and access **Argo CD** on Kubernetes
- How to define and deploy an **Argo CD Application**
- How to create **image pull secrets** for GHCR
- How GitOps enables automated sync, self-heal, and pruning

---

## Install Argo CD

Add the Argo Helm repo and install Argo CD into the `argocd` namespace:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
kubectl create namespace argocd
helm install argocd argo/argo-cd --namespace argocd --version 7.7.0
```

---

## Access the Argo CD UI

Port-forward the Argo CD server service:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:80
```

Then open: http://localhost:8080

---

## Retrieve Admin Credentials

Fetch the initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

---

## Create an Argo CD Application

Below is an example `Application` manifest you can apply to let Argo CD deploy a GitOps repository.

> Replace `YOUR_USERNAME` and repository details as per your setup.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: grade-submission-api
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/YOUR_USERNAME/grade-api-gitops.git
    targetRevision: HEAD
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply it:

```bash
kubectl apply -f application.yaml
```

---

## Create an Image Pull Secret for GHCR

To allow Kubernetes to pull images from **GitHub Container Registry** (`ghcr.io`), create a Docker registry secret:

> Use a GitHub **Personal Access Token (PAT)** with at least `read:packages` permissions.

```bash
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=YOUR_USERNAME \
  --docker-password=YOUR_PAT \
  --namespace=default
```

---

## Author

Created and maintained by **Aniket Purohit** (GitHub: **@aniketpuro**).

---

## License

Add a license file if you plan to distribute or reuse this project publicly (e.g., MIT, Apache-2.0).
