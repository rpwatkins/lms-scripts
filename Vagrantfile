# -*- mode: ruby -*-
# vi: set ft=ruby :

#Scripts (instal & swarm config)
$install_docker_script = <<SCRIPT
echo "Installing dependencies ..."
sudo apt-get update
echo Installing Docker...
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker vagrant
newgrp docker
sudo rm get-docker.sh
SCRIPT

$manager_script = <<SCRIPT
echo Swarm Init...
sudo docker swarm init --listen-addr 172.20.20.11:2377 --advertise-addr 172.20.20.11:2377
sudo docker swarm join-token --quiet worker > /vagrant/worker_token
sudo docker swarm join-token --quiet manager > /vagrant/manager_token
SCRIPT

$manager_join_script = <<SCRIPT
echo Swarm Join...
sudo docker swarm join --token $(cat /vagrant/manager_token) 172.20.20.11:2377
SCRIPT

$worker_script = <<SCRIPT
echo Swarm Join...
sudo docker swarm join --token $(cat /vagrant/worker_token) 172.20.20.11:2377
SCRIPT

#Config
BOX_DISTRIBUTION = "ubuntu/jammy64" #Ubuntu distribution(see https://wiki.ubuntu.com/Releases)
MEMORY = "1024" #Memory allocated to each machine
MANAGERS = 1 #Number of managers
MANAGER_IP = "172.20.20.1" #Manager IP
WORKERS = 3 #Number of workers
WORKER_IP = "172.20.20.10" #Workers IP
CPUS = 2 #Number of CPUs allocated to each machine
VAGRANTFILE_API_VERSION = "2" #Vagrantfile version

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    # Common setup
    config.vm.box = BOX_DISTRIBUTION
    config.vm.synced_folder ".", "/vagrant"
    config.vm.provision "shell",inline: $install_docker_script, privileged: true
    config.ssh.insert_key = false #Prevent SSH key overwriting
    # Setup Manager Nodes
    (1..MANAGERS).each do |i|
        config.vm.define "manager0#{i}" do |manager|
          manager.vm.network :private_network, ip: "#{MANAGER_IP}#{i}"
          manager.vm.hostname = "manager0#{i}"
          if i == 1
            #Only configure port to host for Manager01
            manager.vm.network :forwarded_port, guest: 8080, host: 8080
            manager.vm.network :forwarded_port, guest: 5000, host: 5000
            manager.vm.network :forwarded_port, guest: 9000, host: 9000
            manager.vm.network :forwarded_port, guest: 8000, host: 8000
            manager.vm.network :forwarded_port, guest: 1337, host: 1337
            manager.vm.network :forwarded_port, guest: 27017, host: 27017
            manager.vm.provision "shell",inline: $manager_script, privileged: true
          else
            manager.vm.provision "shell",inline: $manager_join_script, privileged: true
          end
          manager.vm.provider "virtualbox" do |vb|
            vb.name = "manager0#{i}"
            vb.memory = MEMORY
            vb.cpus = CPUS
          end
      end
    end
    # Setup Worker Nodes
    (1..WORKERS).each do |i|
        config.vm.define "worker0#{i}" do |worker|
          worker.vm.provision "shell",inline: $worker_script, privileged: true
          worker.vm.network :private_network, ip: "#{WORKER_IP}#{i}"
          worker.vm.hostname = "worker0#{i}"
          worker.vm.provider "virtualbox" do |vb|
            vb.name = "worker0#{i}"
            vb.memory = MEMORY
            vb.cpus = CPUS
          end
        end
    end
end
