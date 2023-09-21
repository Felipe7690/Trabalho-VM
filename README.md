## Aluno
- Felipe Rafael Gomes 4º per. S.I

# Laboratório de Administração de Redes

Este é um ambiente de laboratório configurado com Vagrant, consistindo de três máquinas virtuais (VMs) para fins educacionais e de treinamento em administração de redes.

## Requisitos

Certifique-se de ter o seguinte software instalado em seu sistema:

- [Vagrant](https://www.vagrantup.com/)
- [VirtualBox](https://www.virtualbox.org/)

## Configuração das Máquinas Virtuais

### VM1 - Servidor Web (Privado)

- **Sistema Operacional:** Ubuntu Server 20.04 LTS
- **Interface de Rede 1 (eth0):** IP Privado Estático (192.168.50.10)
- **Função:** Servidor Web (Apache)
- **Pasta Compartilhada:** `/var/www/html` na máquina host compartilhada com `/var/www/html` na VM1.

```bash
Vagrant.configure("2") do |config|
  config.vm.define "vm1" do |vm1|
    vm1.vm.box = "ubuntu/bionic64"
    vm1.vm.network "private_network", type: "dhcp"
    vm1.vm.network "private_network", type: "static", ip: "192.168.56.10"
    vm1.vm.synced_folder "./html", "/var/www/html"
    vm1.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
    end
    vm1.vm.provision "shell", path: "provision_vm1.sh"
  end
```

### VM2 - Servidor de Banco de Dados (Privado)

- **Sistema Operacional:** Ubuntu Server 20.04 LTS
- **Interface de Rede 1 (eth0):** IP Privado Estático (192.168.50.11)
- **Função:** Servidor de Banco de Dados (MySQL ou PostgreSQL)

```bash
  config.vm.define "vm2" do |vm2|
    vm2.vm.box = "ubuntu/bionic64"
    vm2.vm.network "private_network", type: "static", ip: "192.168.56.11"
    vm2.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
    end
    vm2.vm.provision "shell", path: "provision_vm2.sh"
  end
```

### VM3 - Gateway (Privado DHCP e Pública)

- **Sistema Operacional:** Ubuntu Server 20.04 LTS
- **Interface de Rede 1 (eth0):** IP Privado Estático (192.168.50.12)
- **Interface de Rede 2 (eth1):** IP Público DHCP
- **Função:** Gateway de Rede (Fornecimento de acesso à Internet para VM1 e VM2)

```bash
  config.vm.define "vm3" do |vm3|
    vm3.vm.box = "ubuntu/bionic64"
    vm3.vm.network "private_network", type: "static", ip: "192.168.56.12"
    vm3.vm.network "public_network", type: "bridged", bridge: "wlp3s0"
    vm3.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
    end
    vm3.vm.provision "shell", path: "provision_vm3.sh"
  end
end
```

## Configuração de Rede

As VMs estão configuradas com as seguintes interfaces de rede:

- VM1: IP Privado 192.168.56.10
- VM2: IP Privado 192.168.56.11
- VM3: IP Privado 192.168.56.12 (eth0) e IP Público via DHCP (eth1)

## Scripts de Provisionamento

Cada VM é provisionada com scripts que instalam e configuram os serviços necessários.

VM1
```bash
sudo apt-get update
sudo apt-get upgrade -y

sudo apt-get install apache2 -y

sudo systemctl start apache2
sudo systemctl enable apache2

```

VM2
```bash
sudo apt-get update
sudo apt-get upgrade -y


sudo apt-get install mysql-server -y

sudo systemctl start mysql
sudo systemctl enable mysql

```

VM3
```bash
sudo apt-get update
sudo apt-get -y upgrade

sudo sysctl -w net.ipv4.ip_forward=1

sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE

```

## Uso

Para iniciar as VMs, navegue até o diretório do projeto e execute o seguinte comando:

```bash
vagrant up
```
Para acessar as VMs via SSH, use os seguintes comandos:

VM1 (Servidor Web):

```bash
vagrant ssh vm1
```
VM2 (Servidor de Banco de Dados):

```bash
vagrant ssh vm2
```
VM3 (Gateway):

```bash
vagrant ssh vm3
``````