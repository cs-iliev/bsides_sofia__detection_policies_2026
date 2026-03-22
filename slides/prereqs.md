---
marp: true
theme: default
paginate: true
backgroundColor: #0d1117
color: #e6edf3
style: |
  section {
    font-family: 'JetBrains Mono', 'Fira Code', monospace;
    font-size: 1.1em;
  }
  h1 { color: #58a6ff; border-bottom: 2px solid #21262d; padding-bottom: 0.2em; }
  h2 { color: #79c0ff; }
  h3 { color: #a5d6ff; }
  code { background: #161b22; color: #79c0ff; padding: 0.1em 0.3em; border-radius: 4px; }
  pre { background: #161b22; border: 1px solid #30363d; border-radius: 6px; padding: 1em; }
  pre code { color: #e6edf3; background: none; padding: 0; }
  strong { color: #ffa657; }
  em { color: #7ee787; font-style: normal; }
  table { border-collapse: collapse; width: 100%; }
  th { background: #161b22; color: #79c0ff; padding: 0.4em 0.8em; }
  td { border-top: 1px solid #30363d; padding: 0.4em 0.8em; }
  blockquote { border-left: 4px solid #58a6ff; padding-left: 1em; color: #8b949e; font-style: italic; }
  section.lead h1 { font-size: 2.2em; text-align: center; }
  section.lead h2 { text-align: center; color: #8b949e; }
  section.lead p  { text-align: center; }
  section.divider { display: flex; flex-direction: column; justify-content: center; align-items: center; }
  section.divider h1 { font-size: 2.5em; border: none; }
  .ok   { color: #7ee787; }
  .warn { color: #ffa657; }
---

<!-- _class: lead -->

# Pre-workshop setup
## BSides Sofia 2026 · Policy-as-Detection for Kubernetes

Install everything before the workshop starts.
**~20 minutes. Do it at home.**

---

## What we need

| Tool | Why | Required |
|---|---|---|
| **Docker** | k3d runs Kubernetes inside Docker | ✅ |
| **k3d** | Local Kubernetes cluster | ✅ |
| **kubectl** | Kubernetes CLI | ✅ |
| **Helm** | Installs ArgoCD, Kyverno, Prometheus | ✅ |
| **git** | Clone the repo, push changes | ✅ |
| **jq** | Parse policy reports in the terminal | ✅ |

> *Windows users: install WSL2 first, then follow the Linux steps inside it.*

---

<!-- _class: divider -->

# Step 1
## Docker

---

## Install Docker

**macOS**

Download Docker Desktop from **https://docs.docker.com/desktop/mac/**

Or with Homebrew:

```bash
brew install --cask docker
```

**Linux (Ubuntu / Debian)**

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
# Log out and back in
```

---

## Verify Docker

```bash
docker info
```

Expected: output with `Server Version`, `Storage Driver`, etc.

```bash
docker run --rm hello-world
```

Expected: `Hello from Docker!`

> If Docker Desktop shows a licence dialog — accept it. The free tier is enough.

---

<!-- _class: divider -->

# Step 2
## k3d

---

## Install k3d

k3d runs a full Kubernetes cluster inside Docker containers.
No VM, no cloud account needed.

**macOS**

```bash
brew install k3d
```

**Linux**

```bash
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

**Verify**

```bash
k3d version
```

Expected: `k3d version v5.x.x`

---

## Create the workshop cluster

```bash
git clone https://github.com/jo114ge/bsides_sofia__detection_policies_2026
cd bsides_sofia__detection_policies_2026

CLUSTER_NAME=workshop ./scripts/create-k3d-cluster.sh
```

This creates a cluster named `k3d-workshop` with:
- 1 server node + 1 agent node
- Port 8080 → HTTP loadbalancer
- Port 8443 → HTTPS loadbalancer

**Verify**

```bash
kubectl --context k3d-workshop get nodes
```

Expected: 2 nodes in `Ready` state.

---

<!-- _class: divider -->

# Step 3
## kubectl + Helm

---

## Install kubectl

**macOS**

```bash
brew install kubectl
```

**Linux**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -sL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

**Verify**

```bash
kubectl version --client
kubectl --context k3d-workshop get nodes
```

---

## Install Helm

**macOS**

```bash
brew install helm
```

**Linux**

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**Verify**

```bash
helm version
```

Expected: `version.BuildInfo{Version:"v3.x.x", ...}`

---

<!-- _class: divider -->

# Step 4
## jq + git

---

## Install jq and git

**macOS**

```bash
brew install jq git
```

**Linux**

```bash
sudo apt-get install -y jq git    # Debian / Ubuntu
# or
sudo dnf install -y jq git        # Fedora / RHEL
```

**Verify**

```bash
jq --version        # jq-1.7.x
git --version       # git version 2.x.x
```

---

<!-- _class: divider -->

# Step 5
## Install the workshop stack

---

## One script installs everything

With the cluster running, install ArgoCD, Kyverno and Prometheus:

```bash
cd bsides_sofia__detection_policies_2026
KUBECONTEXT=k3d-workshop ./scripts/install-workshop-stack.sh
```

This takes **3–8 minutes** depending on your connection.

What it does:
```
1. Creates namespaces (argocd, kyverno, monitoring)
2. Installs kube-prometheus-stack  (Prometheus + Grafana + Alertmanager)
3. Installs Kyverno
4. Installs Argo CD
5. Applies Kyverno policies
6. Applies Argo CD Application objects
```

---

## What gets installed

<style scoped>pre { font-size: 0.6em; line-height: 1.4; }</style>

```bash
kubectl --context k3d-workshop get pods -A
```

```
NAMESPACE    NAME                                      READY   STATUS
argocd       argocd-server-xxx                         1/1     Running
argocd       argocd-repo-server-xxx                    1/1     Running
kyverno      kyverno-xxx                               1/1     Running
monitoring   monitoring-grafana-xxx                    2/2     Running
monitoring   monitoring-prometheus-xxx                 2/2     Running
monitoring   alertmanager-monitoring-xxx               2/2     Running
demo         demo-app-xxx                              1/1     Running
```

All pods must be `Running` before the workshop starts.

---

<!-- _class: divider -->

# Step 6
## Verify everything works

---

## Check prerequisites

```bash
KUBECONTEXT=k3d-workshop ./scripts/check-prereqs.sh
```

Expected output (all green):
```
[INFO] Found k3d: /usr/local/bin/k3d
[INFO] Found kubectl: /usr/local/bin/kubectl
[INFO] Found docker: /usr/local/bin/docker
[INFO] Checking API server connectivity
[INFO] Kyverno CRDs detected
[INFO] Argo CD CRDs detected
[INFO] Prometheus Operator CRDs detected
[INFO] Prerequisite check completed
```

If anything fails — fix it before the workshop.

---

## Open the UIs

Start these port-forwards in **two separate terminals** and keep them open:

**Argo CD** — http://localhost:8080

```bash
kubectl --context k3d-workshop -n argocd \
  port-forward svc/argocd-server 8080:80
```

**Grafana** — http://localhost:3000

```bash
kubectl --context k3d-workshop -n monitoring \
  port-forward svc/monitoring-grafana 3000:80
```

---

## Login credentials

**Argo CD** — http://localhost:8080

Username: `admin`

```bash
kubectl --context k3d-workshop -n argocd \
  get secret argocd-initial-admin-secret \
  -o jsonpath='{.data.password}' | base64 -d && echo
```

---

**Grafana** — http://localhost:3000

Username: `admin` · Password: `workshop`

---

## Final verification

**Argo CD** — all three applications should show **Synced · Healthy**:
- `demo-app`
- `kyverno-policies`
- `observability`

**Grafana** — go to Dashboards → verify the Kyverno dashboard loads.

**Policy reports** — should be empty (clean baseline):

```bash
kubectl --context k3d-workshop get policyreport -A -o json | \
  jq '.items[].results[] | select(.result=="fail")'
# Expected: no output
```

---

<!-- _class: divider -->

# Troubleshooting

---

## Pod stuck in Pending or CrashLoopBackOff

```bash
kubectl --context k3d-workshop get pods -A | grep -v Running
kubectl --context k3d-workshop describe pod <pod-name> -n <namespace>
```

Most common causes:
- Docker Desktop does not have enough RAM → give it **at least 4 GB**
- Image pull failed → check your internet connection
- Port already in use → change `HTTP_PORT` in `create-k3d-cluster.sh`

---

## kubectl: context not found

```bash
k3d kubeconfig merge workshop --kubeconfig-switch-context
kubectl config get-contexts
```

---

## Cluster already exists

```bash
# Delete and recreate
k3d cluster delete workshop
CLUSTER_NAME=workshop ./scripts/create-k3d-cluster.sh
```

---

## Start fresh

```bash
KUBECONTEXT=k3d-workshop ./scripts/uninstall-workshop-stack.sh
KUBECONTEXT=k3d-workshop ./scripts/install-workshop-stack.sh
```

---

<!-- _class: lead -->

## Ready?

All pods Running · Argo CD Synced · Grafana loads · Policy reports empty

**See you at BSides Sofia.**

```
https://github.com/jo114ge/bsides_sofia__detection_policies_2026
```
