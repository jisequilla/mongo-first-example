# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos/7"
  
  config.vm.synced_folder ".", "/vagrant", type: "virtualbox", disabled: false,
    rsync__exclude: ".git/"

  $script_generate_ssh_key = <<SCRIPT
	echo Updateing credentials
	if [ ! -f /home/vagrant/.ssh/id_rsa ]; then
	      ssh-keygen -t rsa -N "" -f /home/vagrant/.ssh/id_rsa
	fi
         cp /home/vagrant/.ssh/id_rsa.pub /vagrant/control.pub
         cat << 'SSHEOF' > /home/vagrant/.ssh/config
	 Host *
	 StrictHostKeyChecking no
	 UserKnownHostsFile=/dev/null
SSHEOF
        chown -R vagrant:vagrant /home/vagrant/.ssh/
SCRIPT

  $script_copy_key = 'cat /vagrant/control.pub >> /home/vagrant/.ssh/authorized_keys'

  $script_install_tools = <<SCRIPT
    	echo Installin ansible.
        
		#
		# Update and install basic linux programs for development
		#
		sudo yum update -y
		sudo yum install -y wget
		sudo yum install -y curl
		sudo yum install -y vim
		sudo yum install -y git
		sudo yum install -y build-essential
		sudo yum install -y unzip
		sudo yum install -y epel-release
		sudo yum install -y net-tools
		sudo yum install -y ansible 
		sudo yum install -y docker 
		sudo yum install -y docker-compose 
				
		sudo systemctl start docker.service
		
		sudo groupadd docker
		
		sudo usermod -aG docker vagrant
		
		sudo docker pull mongo
		
		mkdir -p ~/second/data
		
		mkdir -p ~/fourth/data
		
		
		sudo docker run -p 27017:27017 --name my-first-mongo -d mongo
		
		sudo docker run -p 37010:27017 --name -v /home/vagrant/second/data:/data/db my-second-mongo -d mongo
		
		sudo docker run -p 47010:27017 --name my-third-mongo -d mongo
		
		sudo docker run -p 57010:27017 --name -v /home/vagrant/fourth/data:/data/db my-fourth-mongo -d mongo
		
		
		
SCRIPT


   $script_install_mongo = <<SCRIPT
		
	 cat << 'SSHEOF' > /etc/yum.repos.d/mongodb.repo
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
SSHEOF

		sudo yum install -y mongodb-org
		
		sudo service mongod start
		
		sudo chkconfig mongod on
		
		sudo service mongod status

		echo "OK"
SCRIPT


	# Define a new master hosts.
	config.vm.define "mongo_server", primary: true do |mongo_server|	
        mongo_server.vm.network "private_network", ip: "192.168.0.10"
        mongo_server.vm.hostname = "master.mongo.io"
        #mongo_server.vm.provision "file", source: "./ansible/" , destination: "$HOME/ansible/"
        mongo_server.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "8184"]
            vb.customize ["modifyvm", :id, "--cpus", "2"]
            vb.name = "mongo"
        end
        mongo_server.vm.provision "shell", inline: $script_install_tools
        mongo_server.vm.provision 'shell', inline: $script_generate_ssh_key
		#mongo_server.vm.provision 'shell', inline: $script_install_mongo
	end
	
	# Define a new master hosts.
	config.vm.define "mongo_client", primary: true do |mongo_client|	
        mongo_client.vm.network "private_network", ip: "192.168.0.20"
        mongo_client.vm.hostname = "client.mongo.io"
        #mongo_server.vm.provision "file", source: "./ansible/" , destination: "$HOME/ansible/"
        mongo_client.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "8184"]
            vb.customize ["modifyvm", :id, "--cpus", "2"]
            vb.name = "mongo_client"
        end
       # mongo_server.vm.provision "shell", inline: $script_install_tools
        mongo_client.vm.provision 'shell', inline: $script_copy_key
		mongo_client.vm.provision 'shell', inline: $script_install_mongo
	end


end
