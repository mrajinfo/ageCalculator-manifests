# ageCalculator-manifests

Kubernetes manifests for [mrajinfo/ageCalculator](https://github.com/mrajinfo/ageCalculator), deployed by Argo CD.

On every push to `main`, CI in the app repo builds `mrajinfo/agecalculator-frontend` and `mrajinfo/agecalculator-backend`, pushes them to Docker Hub tagged with the short commit SHA, and commits those image references to `k8s/frontend.yaml` and `k8s/backend.yaml` here. Argo CD sees the commit and rolls it out.

| Path | What it is |
| --- | --- |
| `k8s/` | The app: nginx frontend, Express backend, MongoDB, and the Grafana dashboard |
| `argocd/agecalculator-app.yaml` | Argo CD app that syncs `k8s/` into the `agecalculator` namespace |
| `argocd/monitoring-app.yaml` | Argo CD app that installs kube-prometheus-stack (Prometheus and Grafana) into `monitoring` |

## Set up on Minikube

```sh
minikube start --cpus=4 --memory=6g
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.6.1/manifests/install.yaml
kubectl apply -f argocd/
```

## Open the UIs

Each command keeps running; use one terminal per UI.

```sh
kubectl -n agecalculator port-forward svc/frontend 8080:80                         # app         http://localhost:8080
kubectl -n argocd port-forward svc/argocd-server 8443:443                          # Argo CD     https://localhost:8443
kubectl -n monitoring port-forward svc/monitoring-grafana 3000:80                  # Grafana     http://localhost:3000
kubectl -n monitoring port-forward svc/monitoring-kube-prometheus-prometheus 9090  # Prometheus  http://localhost:9090
```

Both logins use the user `admin`. The passwords:

```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o go-template='{{.data.password | base64decode}}'
kubectl -n monitoring get secret monitoring-grafana -o go-template='{{index .data "admin-password" | base64decode}}'
```

In Grafana, open **Dashboards → Age Calculator**.
