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
  if !Vagrant.has_plugin?("sshkey")
    puts 'Sorry, we need just one more dependency'
    puts "Need to install the ruby sshkey gem inside vagrant"
    puts "vagrant plugin install sshkey"
    exit
  end

  vagrant_dir = File.expand_path(File.dirname(__FILE__))
  
  require 'yaml'
  require 'sshkey'

  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--memory", 256]
    v.customize ["modifyvm", :id, "--cpus", 1]
  end

  config.vm.define :master do |master|
    master.vm.box = "precise64"
    master.vm.box_url = "http://files.vagrantup.com/precise64.box"
    master.vm.host_name = "master"
    master.vm.network :private_network, ip: "192.168.2.2"
    master.vm.network :public_network, :bridge => "en0"
    master.vm.synced_folder "./roots/", "/srv/"
    
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

  minions = 3
  minions.times do |i|
    id = "minion#{i}"
    ip = "192.168.2.1#{i}"
    
    pillar = YAML.load_file( vagrant_dir + '/roots/pillar/tinc.sls')

    key = SSHKey.generate
    if id == 'minion0'
      ct = nil
    else
      ct = 'minion0'
    end
    
    if !pillar['tinc']['rnet']
      pillar['tinc']['rnet'] = {}
    end
    if !pillar['tinc']['rnet'].key?(id)
      pillar['tinc']['rnet'][id] =  { 'RSAPublicKey' => key.public_key,
                                      'RSAPrivateKey' => key.private_key,
                                      'host_config' => { 'Address' => ip,
                                                         'Subnet' => "192.168.3.1#{i}/32" },
                                      'tinc_config' => { 'Name' => id,
                                                         'ConnectTo' => ct,
                                                         'Interface' => 'tun0',
                                                         'AddressFamily' => 'ipv4' },
                                      'tinc_up' => "ifconfig $INTERFACE 192.168.3.1#{i} netmask 255.255.255.0",
                                      'tinc_down' => 'ifconfig $INTERFACE down'}
      
      File.open(vagrant_dir + '/roots/pillar/tinc.sls','w') do |f| 
        f.write pillar.to_yaml
      end
    end
    
    minion_conf = { 'master' => '192.168.2.2',
                    'id' => id,
                    'file_client' => 'remote' }
    File.open(vagrant_dir + '/' + id,'w') do |f|
      f.write minion_conf.to_yaml
    end
    
    # exit
    
    config.vm.define id do |minion|
      minion.vm.box = "precise64"
      minion.vm.box_url = "http://files.vagrantup.com/precise64.box"
      minion.vm.host_name = "minion#{i}"
      minion.vm.network :private_network, ip: ip
      minion.vm.network :public_network, :bridge => "en0"
      
      minion.vm.provision :salt do |salt|
        salt.run_highstate = true
        salt.minion_config = "./minion#{i}"
        salt.minion_key = "./keys/minion#{i}.pem"
        salt.minion_pub = "./keys/minion#{i}.pub"
      end
    end
  end
end
