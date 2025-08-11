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
- **Traefik**: Ingress controller
- **Sample App**: Hello World nginx

## Adding Applications

Add YAML files to `clusters/pi/apps/` and commit to git. Flux will automatically deploy them.

## Management Commands

```bash
# Check cluster
kubectl get nodes
kubectl get pods -A

# Check GitOps
flux get sources git
flux get kustomizations

# Check apps
kubectl get pods -n hello-world
```
