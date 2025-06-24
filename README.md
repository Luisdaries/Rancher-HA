
# Levantar Rancher en HA con 3 nodos master de K8s (Ubuntu 22.04)

## Requisitos previos

* Sistema operativo Linux (recomendado: Rocky o Ubuntu Server).
* Acceso con usuario con privilegios de `sudo`.
* Conexión entre nodos por red privada (ej. 10.98.56.x).
* Resolución de nombres mediante archivo `hosts` o DNS interno.
* Token compartido para los nodos (`K3S_TOKEN=SECRET`).
* Herramientas disponibles: `curl`, `helm`, `kubectl`.

---

# Levantar los nodos con vagrant


## Paso a paso: Nodo principal (10.98.56.128)

### 1. Instalar Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

### 2. Instalar K3s en el nodo principal

```bash
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server \
  --cluster-init \
  --tls-san=10.98.56.128 \
  --tls-san=adan-gomez.online
```

### 3. Configurar `kubectl`

```bash
mkdir -p $HOME/.kube
sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verificar estado del nodo:

```bash
kubectl get nodes
```

---

## 4. Instalar Cert-Manager

### Crear namespace

```bash
kubectl create namespace cert-manager
```

### Agregar repositorio de Helm

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

### Instalar Cert-Manager con CRDs

```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.18.0 \
  --set crds.enabled=true
```

Verificar los pods:

```bash
kubectl get pods -n cert-manager
```

---

## 5. Instalar Rancher

### Crear namespace

```bash
kubectl create namespace cattle-system
```

### Agregar repositorio de Rancher

```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update
```

### Instalar Rancher

```bash
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=adan-gomez.online \
  --set bootstrapPassword=admin
```

Esperar que se despliegue Rancher:

```bash
kubectl -n cattle-system rollout status deploy/rancher
```

---

## 6. Verificar el servicio

```bash
kubectl get svc --all-namespaces -o wide
```

---

## 7. Configurar DNS local

Agregar en `/etc/hosts` de cada nodo o configurar en tu DNS:

```
10.98.56.128 k3s.mydomain.com
```

---

## Paso a paso: Nodos secundarios (10.98.56.129, 10.98.56.130, ...)

Ejecutar en cada nodo adicional:

```bash
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server \
  --server https://10.98.56.128:6443 \
  --tls-san=10.98.56.129 \
  --tls-san=adan-gomez.online
```

Nota: Reemplaza `--tls-san` con la IP correspondiente de cada nodo si deseas exponerlo.

---

## Validaciones

* `kubectl get nodes`: Verifica que todos los nodos estén en estado `Ready`.
* `kubectl get pods -A`: Asegúrate de que todos los pods estén en `Running` o `Completed`.
* Accede a la interfaz web: `https://rancher.my.org`

---

## Opcional: Reiniciar Cert-Manager

En caso de necesitar reinstalar cert-manager:

```bash
helm uninstall cert-manager -n cert-manager
kubectl delete crds -l app.kubernetes.io/name=cert-manager
```


# Referencias

- [Nginx LB](https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/infrastructure-setup/nginx-load-balancer)