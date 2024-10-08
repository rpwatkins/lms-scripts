# -*- mode: ruby -*-
# vi: set ft=ruby :

#Scripts (instal & swarm config)
$install_docker_script = <<SCRIPT
export DEBIAN_FRONTEND=noninteractive
echo "Adding docker repo"
sudo apt-get -y update
sudo apt-get -y install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
echo 'installing docker'
sudo apt-get -y update
sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
echo "setup docker permissions"
sudo groupadd docker
sudo usermod -aG docker "$USER"
echo "enable/start docker/container service"
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
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
BOX_DISTRIBUTION = "gusztavvargadr/ubuntu-server-2404-lts" #Ubuntu distribution(see https://wiki.ubuntu.com/Releases)
MEMORY = "1024" #Memory allocated to each machine
MANAGERS = 1 #Number of managers
MANAGER_IP = "172.20.20.1" #Manager IP
WORKERS = 0 #Number of workers
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
          manager.vm.network :private_network, ip: "192.168.56.10"
          manager.vm.hostname = "manager0#{i}"
          if i == 1
            #Only configure port to host for Manager01
            manager.vm.network :forwarded_port, guest: 9120, host: 9120
            #manager.vm.network :forwarded_port, guest: 27017, host: 27017
            #manager.vm.provision "shell",inline: $manager_script, privileged: true
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