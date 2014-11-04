# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# choose how many machines the cluster will contain
N_VMS = 4

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "ubuntu/precise64"
  # use a minimal amount of RAM for each node to avoid overwhelming the host
  config.vm.provider "virtualbox" do |v|
    v.memory = 256
    v.cpus = 1
  end
  config.vm.network "private_network", type: "dhcp"

  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http     = "http://cache1a.internal.sanger.ac.uk:3128/"
    config.proxy.https    = "https://cache1a.internal.sanger.ac.uk:3128/"
    config.proxy.no_proxy = "localhost,127.0.0.1"
  end

  hosts_file = []
  (1..N_VMS).each do |i|
    config.vm.define vm_name = "node#{i}" do |config|
      config.vm.hostname = vm_name
      ip = "192.168.0.#{100+i}"
      config.vm.network "private_network",
        ip: ip,
        virtualbox__intnet: "clusternet"
    end
  end

  script = <<-SCRIPT
    set -x
    if [[ ! -e /etc/.provisioned ]]; then
      rm /etc/hosts
      for i in {1..N_VMS}; do
        ip="192.168.0.$((100+i))"
        echo "$ip node$i" >> /etc/hosts

      done
      wget -q https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant -O id_rsa
      install -m 600 -o vagrant -g vagrant id_rsa /home/vagrant/.ssh/
      sudo apt-get -y install openmpi1.5-bin libopenmpi1.5-dev
      touch /etc/.provisioned
    fi
  SCRIPT
  script.sub! 'N_VMS', N_VMS.to_s
  config.vm.provision "shell", inline: script

end
