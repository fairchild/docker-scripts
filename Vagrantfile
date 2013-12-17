# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

$bootstrap_script=<<SCRIPT
#!/bin/bash
if [ -f /var/log/bootstrapped ];then
  exit 0
fi

set -x
set -e

export DEBIAN_FRONTEND=noninteractive
wget -q -O - https://get.docker.io/gpg | apt-key add -
echo 'deb http://get.docker.io/ubuntu docker main' > /etc/apt/sources.list.d/docker.list
apt-get update -q
apt-get install -q -y --no-upgrade linux-image-generic-lts-raring
apt-get install -q -y lxc-docker
# usermod -a -G docker vagrant
apt-get install -y puppet vim curl git wget
date >> /var/log/bootstrapped

SCRIPT




Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  # config.vm.box_url = "http://domain.com/path/to/above.box"
  config.vm.box = "precise64docker"
  config.vm.provision "shell", inline: $bootstrap_script
  config.vbguest.auto_update = false  # set to false to disable https://github.com/dotless-de/vagrant-vbguest, if installed
  
  # config.vm.provider :digital_ocean do |provider|
  #   config.ssh.private_key_path = "~/.ssh/id_rsa"
  #   config.ssh.username = ENV['USER']
  #   config.vm.box = "digital_ocean"
  #   config.vm.box_url = "https://github.com/smdahlen/vagrant-digitalocean/raw/master/box/digital_ocean.box"
  #   
  #   provider.client_id = ENV['DIGITAL_OCIAN_CLIENT_KEY']
  #   provider.api_key = ENV['DIGITAL_OCIAN_API_KEY']
  #   provider.image = "Ubuntu 13.10 x64"
  #   provider.size = '1GB'
  #   provider.private_networking = false
  #   provider.ssh_key_name = ENV['USER']
  #   # provider.setup = true # default
  # end
  #   
  # config.vm.define "hobo", primary: true do |hobo|
  #   hobo.vm.hostname = 'hobo'
  #   hobo.vm.synced_folder "./", "/vagrant/"
  #   hobo.vm.provision "docker" do |d|
  #     d.run "ubuntu", cmd: "bash -l", args: "-v '/vagrant:/var/www'"
  #   end
  # end


  # Create a private network, which allows host-only access to the machine using a specific IP.
  config.vm.network :private_network, ip: "192.168.3.80"
  
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network :forwarded_port, guest: 5070, host: 5070
  # config.vm.network :forwarded_port, guest: 8080, host: 8080
  # config.vm.network :forwarded_port, guest: 8081, host: 8081
  # config.vm.network :forwarded_port, guest: 80, host: 8080
  

  # If true, then any SSH connections made will enable agent forwarding so we can git clone from private repos without putting the key on the vm
  config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "./", "/home/vagrant/amplab", :nfs=>false

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  config.vm.provider :virtualbox do |vbox|
    vbox.gui = true # Open VirtualBox GUI terminal during provisioning.
    vbox.customize [ "modifyvm", :id, "--memory", 1024 ]
    vbox.customize [ "modifyvm", :id, "--nictype1", "virtio" ]
    # vbox.customize [ "modifyvm", :id, "--nictype2", "virtio" ]
  end
  
  # config.vm.define :namenode, primary: true do |namenode|
  #   namenode.hostname = 'namenode'
  # end
  
  config.vm.provision "docker" do |d|
    d.pull_images "nickstenning/graphite"
    d.pull_images "parente/ipython-notebook"
    d.pull_images "amplab/spark-master"
    d.pull_images "amplab/dnsmasq-precise"
    d.run 'graphite', args: "-p 80 -p 2003 -p 2004 -p 7002", image: "nickstenning/graphite"
  end
  
  config.vm.provision "shell", inline: "sudo /vagrant/deploy/deploy.sh --i amplab/spark:0.8.0 -w 3 "
  
  

end
