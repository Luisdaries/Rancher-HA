
Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2204"
  config.vm.box_version = "4.3.12"


  # No se simplifica el levantamiento haciendo uso de ciclos o relacionados ya que se requiere la facil comprencion del Ejercicio
  # Caracteristicas del Nodo maestro
  config.vm.define "k3s-master-1" do |master|
    master.vm.hostname = "k3s-master" # Nombre del nodo maestro
    master.vm.network "private_network", ip: "10.98.56.128" # IP del nodo maestro
    master.vm.provider "virtualbox" do |vb|
      vb.memory = "2048" # Memoria RAM del nodo maestro
      vb.cpus = 2 # Numero de CPUs del nodo maestro
    end
    master.vm.provision "shell", inline: <<-SHELL
      apt update
    SHELL
  end
# Caracteristicas del Nodo trabajador 1
  config.vm.define "k3s-master-2" do |worker|
    worker.vm.hostname = "k3s-master-2" # Nombre del nodo trabajador 1
    worker.vm.network "private_network", ip: "10.98.56.129" # IP del nodo trabajador 1
    worker.vm.provider "virtualbox" do |vb|
      vb.memory = "2048" # Memoria RAM del nodo trabajador 1
      vb.cpus = 2 # Numero de CPUs del nodo trabajador 1
    end
    worker.vm.provision "shell", inline: <<-SHELL
      apt update
    SHELL
  end
# Caracteristicas del Nodo trabajador 2
config.vm.define "k3s-master-3" do |worker_2|
  worker_2.vm.hostname = "k3s-master-3" # Nombre del nodo trabajador 2
  worker_2.vm.network "private_network", ip: "10.98.56.130" # IP del nodo trabajador 2
  worker_2.vm.provider "virtualbox" do |vb|
    vb.memory = "2048" # Memoria RAM del nodo trabajador 2
    vb.cpus = 2 # Numero de CPUs del nodo trabajador 2
  end
  worker_2.vm.provision "shell", inline: <<-SHELL
    apt update
  SHELL
end
end