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
kubectl apply -f clusters/pi/system/gitops-source.yaml
```

## What's Included

- **K3s**: Lightweight Kubernetes
- **Flux**: GitOps operator  
- **Traefik**: Ingress controller with SSL (included with K3s)
- **Cert-Manager**: Automatic Let's Encrypt certificates
- **Portainer**: Kubernetes management UI

## DNS Configuration ✅

Domain is configured and working:
- Router port forwarding: 80/443 → 192.168.1.150  
- SSL certificates automatically issued by Let's Encrypt
- Applications accessible via `*.ulyssetassidis.fr` subdomains

## Adding Applications

### Method 1: Add to this repository
Add YAML files to `clusters/pi/apps/` and commit to git. Flux will automatically deploy them.

### Method 2: Deploy from another repository

1. **Create k8s manifests in your app repository:**
   ```bash
   # In your app repo (e.g., ~/code/my-app)
   mkdir k8s/
   # Add your deployment.yaml, service.yaml, ingress.yaml, etc.
   # Include kustomization.yaml listing all resources
   ```

2. **Add GitOps source in this repo:**
   ```bash
   # Create clusters/pi/system/my-app-source.yaml
   cat > clusters/pi/system/my-app-source.yaml << EOF
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
   EOF
   ```

3. **Add to system kustomization:**
   ```bash
   echo "- my-app-source.yaml" >> clusters/pi/system/kustomization.yaml
   ```

4. **Commit and deploy:**
   ```bash
   git add . && git commit -m "Add my-app" && git push
   # Or apply directly: kubectl apply -f clusters/pi/system/my-app-source.yaml
   ```

5. **Check deployment (automatic within 1 minute):**
   ```bash
   # Flux checks every 1 minute automatically
   # To check immediately without waiting:
   flux reconcile source git pi-aws
   
   # Monitor status:
   flux get sources git
   flux get kustomizations
   kubectl get pods -n your-namespace
   ```

**Example**: The monitoring stack (Grafana/Prometheus) uses this pattern via `pi-grafana-source.yaml`.

## Management Commands

```bash
# Check cluster
kubectl get nodes
kubectl get pods -A

# Check GitOps
flux get sources git
flux get kustomizations

# Check apps and certificates
kubectl get pods -n portainer
kubectl get certificates -A  
kubectl get ingress -A

# Test applications (if any are deployed)
# Example: curl https://portainer.ulyssetassidis.fr
```
