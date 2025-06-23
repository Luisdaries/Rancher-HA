Levantar Rancher en HA con 3 nodos master de k8s con ubuntu 22.04

pass: wl7sE94z89w3GwzX



Principal
    9  curl -sfL https://get.k3s.io | sh -s - server   --tls-san 192.168.56.10   --tls-san k3s-master-1   --cluster-init
   10  sudo cat /var/lib/rancher/k3s/server/node-token
   11  sudo kubectl get nodes
   13  sudo nano /etc/rancher/k3s/k3s.yaml
   16  sudo ufw allow 6443/tcp #Api server
   18  sudo cat /etc/rancher/k3s/k3s.yaml 

Adicionales

   12  curl -sfL https://get.k3s.io | K3S_URL=https://192.168.56.10:6443 K3S_TOKEN=K102c03bb62c8b47c74e0aff069936c4551bacddb07fa75761dc21f2b3799583156::server:e9ddbfd17e2aae4d0716c4572ee5ce68 sh -s - server


K8s

kubectl get pods --all-namespaces
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.2/cert-manager.yaml
helm install rancher rancher-stable/rancher   --namespace cattle-system   --set hostname=miapp.local  --set ingress.tls.source=letsEncrypt   --set letsEncrypt.email=tu@email.com   --set letsEncrypt.ingress.class=nginx
kubectl -n cattle-system rollout status deploy/rancher
 kubectl get ingress -n cattle-system


---
RESUMEN

  238  kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml
  239  kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.2/cert-manager.yaml
  240  helm install rancher rancher-latest/rancher   --namespace cattle-system   --set hostname=rancher.midominio.com   --set ingress.tls.source=letsEncrypt   --set letsEncrypt.email=tu@email.com   --set letsEncrypt.ingress.class=nginx
  241  helm install rancher rancher-latest/rancher   --namespace cattle-system   --set hostname=rancher.midominio.com   --set ingress.tls.source=letsEncrypt   --set letsEncrypt.email=tu@email.com   --set letsEncrypt.ingress.class=nginx
  242  helm install rancher rancher/rancher   --namespace cattle-system   --set hostname=rancher.midominio.com   --set ingress.tls.source=letsEncrypt   --set letsEncrypt.email=tu@email.com   --set letsEncrypt.ingress.class=nginx
  243  helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
  244  kubectl create namespace cattle-system
  245  kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/<VERSION>/cert-manager.crds.yaml
  246  helm install rancher rancher-latest/rancher   --namespace cattle-system   --set hostname=rancher.midominio.com   --set ingress.tls.source=letsEncrypt   --set letsEncrypt.email=tu@email.com   --set letsEncrypt.ingress.class=nginx
  247  helm install rancher rancher-stable/rancher   --namespace cattle-system   --set hostname=rancher.midominio.com   --set ingress.tls.source=letsEncrypt   --set letsEncrypt.email=tu@email.com   --set letsEncrypt.ingress.class=nginx
  248  kubectl -n cattle-system rollout status deploy/rancher
  249  kubectl get pods -n cattle-system
  250  kubectl get pods -n cattle-system -i wide
  251  kubectl get pods -n cattle-system -o wide
  252  kubectl get nodes -o wide
  253  kubectl get pods -n cattle-system -i wide
  254  kubectl get pods -n cattle-system -o wide
  255  kubectl get ingress -n cattle-system
  256  kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{"\n"}}'
  257  history