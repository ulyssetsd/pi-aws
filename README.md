# K3s + GitOps on Raspberry Pi

Simple Kubernetes setup with GitOps using K3s and Flux.

## Quick Setup

```bash
# 1. Enable memory cgroups (requires reboot)
sudo cp /boot/firmware/cmdline.txt /boot/firmware/cmdline.txt.backup
echo "$(cat /boot/firmware/cmdline.txt) cgroup_memory=1 cgroup_enable=memory" | sudo tee /boot/firmware/cmdline.txt
sudo reboot

# 2. Install K3s
curl -sfL https://get.k3s.io | sh -
sudo systemctl start k3s

# 3. Setup kubectl
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
sudo chmod 644 /etc/rancher/k3s/k3s.yaml

# 4. Install Flux
curl -s https://fluxcd.io/install.sh | sudo bash
flux install

# 5. Setup GitOps
kubectl apply -f clusters/pi/bootstrap/gitops-source.yaml

# 6. Verify Installation (optional)
# Wait 2-3 minutes for everything to deploy, then check:
kubectl get pods -A                    # Check all pods
flux get kustomizations               # Check GitOps status  
kubectl get certificates -A           # Check SSL certificates
kubectl get ingress -A                # Check web services
```

## What's Included

- **K3s**: Lightweight Kubernetes
- **Flux**: GitOps operator  
- **Traefik**: Ingress controller with SSL (included with K3s)
- **Cert-Manager**: Automatic Let's Encrypt certificates
- **WG-Easy**: WireGuard VPN management interface
- **Portainer**: Kubernetes management UI

## Adding New Applications

To add a new application:

1. Create a single YAML file in `clusters/pi/apps/` using one of the patterns above
2. Add the filename to `clusters/pi/apps/kustomization.yaml`
3. Commit and push - Flux will automatically deploy it!

## Application Patterns

Choose the appropriate pattern for your application:

### Pattern 1: Direct Kubernetes Manifests
Use this for simple applications with all resources defined directly:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app
spec:
  # ... deployment spec
---
apiVersion: v1
kind: Service
metadata:
  name: my-app
  namespace: my-app
spec:
  # ... service spec
```

### Pattern 2: External Git Repository
Use this for applications maintained in separate repositories:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/username/my-app
  ref:
    branch: main
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: my-app
  path: "./k8s"
  prune: true
  wait: true
```

## DNS Configuration ✅

Domain is configured and working:
- Router port forwarding: 80/443 → 192.168.1.150  
- SSL certificates automatically issued by Let's Encrypt
- Applications accessible via `*.ulyssetassidis.fr` subdomains
