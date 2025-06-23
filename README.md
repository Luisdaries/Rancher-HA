Levantar Rancher en HA con 3 nodos master de k8s con ubuntu 22.04

pass: .password file



Principal
    9  curl -sfL https://get.k3s.io | sh -s - server   --tls-san 192.168.56.10   --tls-san k3s-master-1   --cluster-init
   10  sudo cat /var/lib/rancher/k3s/server/node-token
   11  sudo kubectl get nodes
   13  sudo nano /etc/rancher/k3s/k3s.yaml
   16  sudo ufw allow 6443/tcp #Api server
   18  sudo cat /etc/rancher/k3s/k3s.yaml 

Adicionales

   12  curl -sfL https://get.k3s.io | K3S_URL=https://192.168.56.10:6443 K3S_TOKEN=K10c664a2f6cb0c83ec7c766d4075d37b649ac5bcd5d19a810bdefbd8be5daf3d54::server:b56a67693ddd5367bb6a1884feca88ba sh -s - server


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

