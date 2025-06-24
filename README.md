
# Levantar Rancher en HA con 3 nodos master de K8s (Ubuntu 22.04)

## Requisitos previos

* Sistema operativo Linux (recomendado: Rocky o Ubuntu Server).
* Acceso con usuario con privilegios de `sudo`.
* Conexión entre nodos por red privada (ej. 10.98.56.x).
* Resolución de nombres mediante archivo `hosts` o DNS interno.
* Token compartido para los nodos (`K3S_TOKEN=SECRET`).
* Herramientas disponibles: `curl`, `helm`, `kubectl`, `docker`, `docker-compose`.

## Consideraciones

* Tener un certificado SSL verificado y vigente con el nombre de dominio con el que levantaremos el aplicativo
---

# Levantar los nodos con vagrant

1. Instalacion de vagrant local

    Para la instalacion de dicha herramienta se debe de ir al sitio oficial de [hashicorp/vagrant](https://developer.hashicorp.com/vagrant/install) y posteriormente ir a la seccion correspondiente en este caso ubuntu
    ```

    wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

    sudo apt update && sudo apt install vagrant

    ```

2. Instalacion Libvirt / plugin Vagrant
    ```
    # Instalacion de libvirt
    sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils -y

    # Instalacion de librerias adiciones para el plugin de vagrant
    sudo apt-get -y install libxslt-dev libxml2-dev libvirt-dev zlib1g-dev ruby-dev

    # Instalacion Plugin libvirt - Vagrant 
    vagrant plugin install vagrant-libvirt

    # Instalacion del gestor VM
    sudo apt install virt-manager -y
    
    # Agregar usuario al grupo de libvirt y kvm
    sudo adduser [username] libvirt
    sudo adduser [username] kvm
    ```

3. Levantar instancias de ubuntu 22.04
   
   En base a los requerimientos solicitados por la practica se hace uso del archivo **Vagrantfile** en esta documentacion.

   ```
   # En la ruta donde este el archivo **Vagrantfile**
   - vagrant up
   ```


# Conectarse a algun nodo con Vagrant

En relacion a que este levantada nuestra infraestructura con ayuda de vagrant se requiere conectar a algun nodo para poder continuar
 
   ```
   # En la ruta donde este el archivo **Vagrantfile** para ver los nodos corriendo
   - vagrant status
   
   # En la ruta donde este el archivo **Vagrantfile**
   - vagrant ssh < Nombre del Nodo >


   ```


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

## Nodos secundarios (10.98.56.129, 10.98.56.130, ...)

Ejecutar en cada nodo adicional:

```bash
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server \
  --server https://10.98.56.128:6443 \
  --tls-san=10.98.56.129 \
  --tls-san=adan-gomez.online
```

Nota: Reemplaza `--tls-san` con la IP correspondiente de cada nodo si deseas exponerlo.

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

## 8. Balanceador de Carga

En la ruta donde se encuentra el archivo `docker-compose.yml` se ejecutara el siguiente comando:

```
docker compose -f 'docker-compose.yml' up -d --build 
```

El cual levantara un contenedor de nginx el que nos servira para tener nuestro balanceador de carga al puerto 80 y 443 de nuestros nodos.

---

## Validaciones

* `kubectl get nodes`: Verifica que todos los nodos estén en estado `Ready`.
* `kubectl get pods -A`: Asegúrate de que todos los pods estén en `Running` o `Completed`.
* Accede a la interfaz web: `https://adan-gomez.online`

---

# Referencias

- [Docker](https://docs.docker.com/)
- [Rancher](https://ranchermanager.docs.rancher.com/)
- [Nginx LB](https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/infrastructure-setup/nginx-load-balancer)
- [Vagrant Docs](https://developer.hashicorp.com/vagrant/docs)
- [ZeroSSL](https://app.zerossl.com)
- [Exclusive Hosting](https://www.exclusivehosting.net/)