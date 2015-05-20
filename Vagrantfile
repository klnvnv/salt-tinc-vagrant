# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  # config.vm.box = "base"
  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL


  # kln
  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--memory", 256]
    v.customize ["modifyvm", :id, "--cpus", 1]
  end
  # config.vm.provision :shell, :path => ""

  config.vm.define :master do |master|
    master.vm.box = "precise64"
    master.vm.host_name = "master"
    master.vm.network :private_network, ip: "192.168.2.2"
    master.vm.network :public_network, :bridge => "en0"
    master.vm.synced_folder "./roots/", "/srv/"
    # master.vm.synced_folder "./keys/", "/etc/salt/keys"
    
    master.vm.provision :salt do |salt|
      salt.install_master = true
      salt.no_minion = true
      salt.master_key = "./keys/master.pem"
      salt.master_pub = "./keys/master.pub"
      salt.seed_master = {master: "./keys/master.pub",
                          minion0: "./keys/minion0.pub",
                          minion1: "./keys/minion1.pub",
                          minion2: "./keys/minion2.pub"}
      salt.master_config = "./master"
      salt.run_highstate = true
      salt.verbose = true
      salt.bootstrap_options = "-D"
      salt.temp_config_dir = "/tmp"
    end
  end
  
  3.times do |i|
    id = "minion#{i}"
    config.vm.define id do |minion|
      minion.vm.box = "precise64"
      minion.vm.host_name = "minion#{i}"
      minion.vm.network :private_network, ip: "192.168.2.1#{i}"
      minion.vm.network :public_network, :bridge => "en0"
      # minion.vm.synced_folder "./keys/", "/etc/salt/keys"
      
      minion.vm.provision :salt do |salt|
        salt.run_highstate = true
        salt.minion_config = "./minion#{i}"
        salt.minion_key = "./keys/minion#{i}.pem"
        salt.minion_pub = "./keys/minion#{i}.pub"
      end
    end
  end
end
