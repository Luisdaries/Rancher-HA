Levantar Rancher en HA con 3 nodos master de k8s con ubuntu 22.04

pass: .password file



Principal
    9  curl -sfL https://get.k3s.io | sh -s - server   --tls-san 10.98.56.128   --cluster-init
   10  sudo cat /var/lib/rancher/k3s/server/node-token
   11  sudo kubectl get nodes
   13  sudo nano /etc/rancher/k3s/k3s.yaml
   16  sudo ufw allow 6443/tcp #Api server
   18  sudo cat /etc/rancher/k3s/k3s.yaml 

Adicionales

   12  curl -sfL https://get.k3s.io | K3S_URL=https://10.98.56.128:6443 K3S_TOKEN=K10ffb0077613cce28efc962abfbe2f04aa186fdb4e5f460cda762de356847e00a5::server:e00ce9d9cbb10d93f40e37820e867c75 sh -s - server


K8s

mkdir -p $HOME/.kube
sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


kubectl get pods --all-namespaces
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.2/cert-manager.yaml
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update
kubectl create namespace cattle-system
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


---

Nodo principal


# 1. Ejecucion del primer nodo 

```bash
curl -sfL https://get.k3s.io | sh -s - server \
  --cluster-init \
  --tls-san 10.98.56.128 \
  --tls-san adan-gomez.online \
  --advertise-address 10.98.56.128 \
  --node-external-ip 10.98.56.128
```

---

# Explicación del comando de instalación inicial de K3s (primer nodo servidor)


- **`curl -sfL https://get.k3s.io`**  
  Descarga de manera silenciosa (`-s`), con fallo inmediato en caso de error (`-f`), y siguiendo redirecciones (`-L`) el script oficial de instalación de K3s desde su repositorio oficial.

- **`sh -s -`**  
  Ejecuta el script descargado directamente en la shell actual.  
  El guion (`-`) indica que los argumentos posteriores serán pasados al script como parámetros.

- **`server`**  
  Especifica que el nodo donde se ejecuta el comando actuará como **servidor del clúster (control plane)**.

- **`--cluster-init`**  
  Inicializa un nuevo clúster K3s.  
  Este parámetro **solo se utiliza en el primer nodo maestro** que se encarga de crear el clúster por primera vez.

- **`--tls-san adan-gomez.online`**  
  Agrega un **Subject Alternative Name (SAN)** al certificado TLS del API Server para aceptar conexiones seguras usando el dominio `adan-gomez.online`.

- **`--tls-san 10.98.56.128`**  
  También agrega la IP `10.98.56.128` como SAN en el certificado TLS, permitiendo el acceso al API Server desde esa IP sin errores de certificado.

- **`--advertise-address 10.98.56.128`**  
  Define la dirección IP que el API Server usará para **anunciarse a otros componentes del clúster**, como los nodos worker o etcd.  
  Es especialmente útil si el nodo tiene múltiples interfaces de red o direcciones IP.

- **`--node-external-ip 10.98.56.128`**  
  Establece la IP externa que otros nodos o herramientas (como Rancher) pueden usar para comunicarse con este nodo.  
  Es útil en entornos con NAT, red en puente (*bridged*), o máquinas virtuales (por ejemplo, en Vagrant).



# Nodos adicionales 

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://10.98.56.129:6443 K3S_TOKEN=K104be600f986bf741e24c001c5f107d65a4b7cd77367203025a47e1014df2d9e0a::server:acc2784508b33197e42a5c4e9f4b3038 \
sh -s - server
````

## 📝 Explicación de cada parte

* **`curl -sfL https://get.k3s.io`**
  Descarga de manera silenciosa (`-s`), con falla rápida (`-f`) y seguimiento de redirecciones (`-L`), el script oficial de instalación de K3s desde internet.

* **`K3S_URL=https://192.168.56.10:6443`**
  Define la URL del API Server del clúster al que este nodo debe conectarse.
  En este ejemplo, la IP del nodo principal es `192.168.56.10` y escucha en el puerto `6443`.

* **`K3S_TOKEN=...`**
  Token secreto necesario para autenticar al nodo al momento de unirse al clúster.
  Se obtiene desde el primer nodo del clúster en la ruta:
  `/var/lib/rancher/k3s/server/node-token`.

* **`sh -s - server`**
  Ejecuta el script descargado con los argumentos indicados, instalando K3s como un **nodo servidor adicional**, no como worker.





---
Nodo principal
    1  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
    2  helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
    3  kubectl create namespace cattle-system
    4  curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server     --cluster-init     --tls-san=10.98.56.128
    5  kubectl get nodes
    6  sudo kubectl get nodes
    7  kubectl create namespace cattle-system
    8  mkdir -p $HOME/.kube
    9  sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
   10  sudo chown $(id -u):$(id -g) $HOME/.kube/config
   11  kubectl create namespace cattle-system
   12  sudo 
   13  kubectl create namespace cattle-system
   14  sudo kubectl create namespace cattle-system
   15  kubectl create namespace cert-manager
   16  sudo kubectl create namespace cert-manager
   17   helm repo add jetstack https://charts.jetstack.io
   18  helm repo update
   19  helm install   cert-manager jetstack/cert-manager   --namespace cert-manager   --version v1.7.1
   20  kubectl get pods --namespace cert-manager
   21  sudo kubectl get pods --namespace cert-manager
   22   helm install rancher rancher-stable/rancher   --namespace cattle-system   --set hostname=rancher.example.com
   23  helm install rancher rancher-estable/rancher   --namespace cattle-system   --set hostname=rancher.my.org   --set bootstrapPassword=admin
   24  helm install rancher rancher-stable/rancher   --namespace cattle-system   --set hostname=rancher.my.org   --set bootstrapPassword=admin
   25  helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
   26  kubectl create namespace cattle-system
   27  sudo kubectl create namespace cattle-system
   28  helm install rancher rancher-stable/rancher   --namespace cattle-system   --set hostname=rancher.my.org   --set bootstrapPassword=admin
   29  kubectl get pods -n cert-manager
   30  sudo kubectl get pods -n cert-manager
   31  # Agrega el repositorio de cert-manager
   32  helm repo add jetstack https://charts.jetstack.io
   33  helm repo update
   34  # Crea el namespace si no existe
   35  kubectl create namespace cert-manager
   36  # Instala cert-manager con sus CRDs
   37  helm install cert-manager jetstack/cert-manager   --namespace cert-manager   --set installCRDs=true
   38  sudo helm install cert-manager jetstack/cert-manager   --namespace cert-manager   --set installCRDs=true
   39  helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
   40  helm repo update
   41  helm install rancher rancher-stable/rancher   --namespace cattle-system   --set hostname=rancher.my.org   --set bootstrapPassword=admin
   42  helm uninstall cert-manager -n cert-manager
   43  kubectl delete crds -l app.kubernetes.io/name=cert-manager
   44  sudo kubectl delete crds -l app.kubernetes.io/name=cert-manager
   45  kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/<VERSION>/cert-manager.crds.yaml
   46  helm repo add jetstack https://charts.jetstack.io --force-update
   47  helm install   cert-manager jetstack/cert-manager   --namespace cert-manager   --create-namespace   --version v1.18.0   --set crds.enabled=true
   48  helm install rancher rancher-stable/rancher   --namespace cattle-system   --set hostname=rancher.my.org   --set bootstrapPassword=admin
   49  kubectl -n cattle-system rollout status deploy/rancher
   50  sudo kubectl -n cattle-system rollout status deploy/rancher
   51  kubectl -n cattle-system rollout status deploy/rancher
   52  sudo kubectl -n cattle-system rollout status deploy/rancher
   53  kubectl get svc --all-namespaces -o wide
   54  sudo kubectl get svc --all-namespaces -o wide

Nodos externos:
curl -sfL https://get.k3s.io | K3S_TOKEN=K10773d67a905138e0cee6a3f8844a642c5ff3b5477e06e5cb0e0c17377ffe52ece::server:0c3bc785363bd8685f1626983a796404 sh -s - server     --server https://10.98.56.128:6443     --tls-san=10.98.56.129
DNS local
10.98.56.128 rancher.my.org

   ---

