# -*- mode: ruby -*-
# vi: set ft=ruby :

# The host name (you should also change it in the ansible/vars/globals.yml)
HOSTNAME = 'site.dev'
ALIASES = %w(api.site.dev)

# If you want to use 'rsync' synchronization on Linux systems
# Ubuntu: apt-get -y install rsync
# CentOS: yum -y install rsync
# --------
# If you want to use 'nfs' synchronization on Linux systems
# Ubuntu: apt-get -y install nfs-kernel-server nfs-common
# CentOS: yum -y install nfs-utils nfs-utils-lib

# Synchronization type may be 'nfs' or 'rsync'
SYNC_TYPE = 'nfs'

# I don't think you want to change it... In that case you should also change it in ansible/inventory
IP_ADDRESS = '10.100.100.100'

# Check to determine whether we're on a windows or linux/os-x host,
# later on we use this to launch ansible in the supported way
# source: https://stackoverflow.com/questions/2108727/which-in-ruby-checking-if-program-exists-in-path-from-ruby
def which(cmd)
    exts = ENV['PATHEXT'] ? ENV['PATHEXT'].split(';') : ['']

    ENV['PATH'].split(File::PATH_SEPARATOR).each do |path|
        exts.each { |ext|
            exe = File.join(path, "#{cmd}#{ext}")
            return exe if File.executable? exe
        }
    end

    return nil
end

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

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
  #config.vm.network :forwarded_port, guest: 80, host: 4567
  config.vm.hostname = HOSTNAME
  config.vm.network :private_network, :ip => IP_ADDRESS

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

  config.ssh.forward_agent = true


  # Setup directories synchronization mode
  if Vagrant::Util::Platform.windows?
    sync_options = {
      :mount_options => ["dmode=775", "fmode=775"],
      :owner => 'vagrant',
      :group => 'vagrant'
    }
  else
    if SYNC_TYPE == 'rsync'
      sync_options = {
        :type => 'rsync',
        :mount_options => ["dmode=775", "fmode=775"],
        :owner => 'vagrant',
        :group => 'vagrant',
        :rsync__args => ['--verbose', '--archive', '-z'],
        :rsync__exclude => ['.git/', '.vagrant/', '.idea/'],
        :rsync__auto => true
      }
    else
      sync_options = {
        :nfs => { :mount_options => ["dmode=775", "fmode=775", "actimeo=1", "lookupcache=non", "rw", "vers=3", "tcp", "fsc"] }
      }
    end
  end

  config.vm.synced_folder ".", "/var/www/vagrant", sync_options


  # Update "host" file for host/guest machines
  if Vagrant.has_plugin? 'vagrant-hostmanager'
    config.hostmanager.manage_host = true
    config.hostmanager.enabled = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true

    if ALIASES.any?
      config.hostmanager.aliases = ALIASES
    end
  end


  # Configure VirtualBox provider
  config.vm.provider :virtualbox do |vb|
    host = RbConfig::CONFIG['host_os']

    # Give VM 1/2 system memory & access to all cpu cores on the host
    if host =~ /darwin/
      cpus = `sysctl -n hw.ncpu`.to_i
      # sysctl returns Bytes and we need to convert to MB
      memory = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 2
    elsif host =~ /linux/
      cpus = `nproc`.to_i
      # meminfo shows KB and we need to convert to MB
      memory = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 2
    else # sorry Windows folks, I can't help you
      cpus = 2
      memory = 2048
    end

    vb.name = HOSTNAME
    vb.customize [ "modifyvm", :id, "--cpus", cpus ]
    vb.customize [ 'modifyvm', :id, '--memory', memory ]
    vb.customize [ 'modifyvm', :id, '--natdnshostresolver1', 'on']
    vb.customize [ 'modifyvm', :id, '--natdnsproxy1', 'on' ]
  end

    # Run Host Manager provision
  config.vm.provision :hostmanager


   config.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/playbook.yml"
      ansible.inventory_path = "ansible/inventory"
      ansible.limit = 'all'
   end

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
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
  #config.vm.provision :shell, path: "bootstrap.sh"

end
