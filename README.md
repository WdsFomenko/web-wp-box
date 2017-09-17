Web-box Project
================


### Backend installation
* Install Virtual Box (the ~5.0 version is preferred) https://www.virtualbox.org/wiki/Downloads
* Install Vagrant (>=1.7.4 version) https://www.vagrantup.com/downloads.html
* Install Vagrant HostManager plugin (`vagrant plugin install vagrant-hostmanager`)
* Install Ansible (>=2.0.0 version) http://docs.ansible.com/ansible/intro_installation.html
* If you have either Ubuntu it is preferred to install nfs-karnel-service (`apt-get install nfs-kernel-server`) to your host machine
* If you install the project from the CodeTiburon office, you need copy `./ansible/vars/locals.yml.dist` to `./ansible/vars/locals.yml` and put there your GitHub token, due to the large number of composer packages installations from the same IP address.
* Run `vagrant up`

