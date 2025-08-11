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
- **Traefik**: Ingress controller with SSL
- **Cert-Manager**: Automatic Let's Encrypt certificates
- **Sample App**: Hello World nginx at `hello.ulyssetassidis.fr`

## DNS Configuration Required

Point your domain to your Raspberry Pi:
```
hello.ulyssetassidis.fr A 192.168.1.150
```

For external access, configure your router to forward ports 80/443 to your Pi.

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

# Check apps and certificates
kubectl get pods -n hello-world
kubectl get certificates -A
kubectl get ingress -A

# Test the application
curl https://hello.ulyssetassidis.fr
```
