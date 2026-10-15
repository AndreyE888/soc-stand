# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'

Vagrant.configure("2") do |config|

  config.vm.box = "bento/ubuntu-24.04"

  config.vm.provider :virtualbox do |virtualbox|
    virtualbox.memory = 2048
    virtualbox.cpus = 2
  end

  config.vm.provision "shell", run: "always", inline: <<-SHELL
    mkdir -p ~root/.ssh
    cp ~vagrant/.ssh/auth* ~root/.ssh 2>/dev/null || true
  SHELL

  # Машина 1: docker
  config.vm.define "docker" do |node|
    node.vm.hostname = "docker"
    node.vm.network "private_network", ip: "192.168.56.10"
    node.vm.provider :virtualbox do |vb|
      vb.name = "docker"
      vb.memory = 2048
      vb.cpus = 2
    end
  end

  # Машина 2: docker-registry
  config.vm.define "docker_registry" do |node|
    node.vm.hostname = "docker-registry"
    node.vm.network "private_network", ip: "192.168.56.11"
    node.vm.provider :virtualbox do |vb|
      vb.name = "docker_registry"
      vb.memory = 2048
      vb.cpus = 2
    end
  end

  # Машина 3: storage
  config.vm.define "storage" do |node|
    node.vm.hostname = "storage"
    node.vm.network "private_network", ip: "192.168.56.12"
    node.vm.provider :virtualbox do |vb|
      vb.name = "storage"
      vb.memory = 2048
      vb.cpus = 2
    end
  end
end
