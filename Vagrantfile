# -*- mode: ruby -*-
# vi: set ft=ruby :

# Install RVM
$installrvmscript = <<SCRIPT
pushd /tmp
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -sSL https://get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh
popd
SCRIPT

# Install Gems from Gemfile
$bundlescript = <<SCRIPT
pushd /srv/git-lfs-s3
bundle
popd
SCRIPT


Vagrant.configure("2") do |config|
	
	# Use this box for VMWare Workstation
	config.vm.box = "bento/ubuntu-16.04"

	# Override box for Virtualbox
	config.vm.provider "virtualbox" do |vb, override|
		override.vm.box = "ubuntu/xenial64"
	end

	# Override box for AWS
	config.vm.provider "aws" do |aws, override|
		override.vm.box = "dummy"
		
		aws.ami = "AMI HERE"
		aws.keypair_name = "KEYPAIR NAME"
		
		override.ssh.username = "ubuntu"
		override.ssh.private_key_path = ""
	end
	
	# Create a forwarded port mapping which allows access to a specific port
	# within the machine from a port on the host machine. In the example below,
	# accessing "localhost:8080" will access port 80 on the guest machine.
	config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"


	# Install RVM
	config.vm.provision "shell", privileged: false, inline: $installrvmscript

	# Setup RVM. We do this from a shell script since we need to be in a "login" shell for this to work.
	config.vm.provision "shell", privileged: false, path: "./setuprvm.sh"
	 
	config.vm.provision "shell", inline: <<-SHELL
	
	
		apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
		apt-get install -y apt-transport-https ca-certificates
		sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger xenial main > /etc/apt/sources.list.d/passenger.list'
		apt-get update
		apt-get install -y nginx git nginx-extras passenger

		# Install Redis
		pushd /tmp
		curl -O http://download.redis.io/redis-stable.tar.gz
		tar xzvf redis-stable.tar.gz
		cd redis-stable
		make
		make install
		mkdir /etc/redis
		cp /vagrant/extras/redis.conf /etc/redis
		cp /vagrant/extras/redis.service /etc/systemd/system
		adduser --system --group --no-create-home redis
		mkdir /var/lib/redis
		chown redis:redis /var/lib/redis
		chmod 770 /var/lib/redis
		systemctl start redis
		systemctl enable redis
		popd


		# Install RVM and the deps required by the app
		# Also, copy over the various config files in /extras required by passenger and nginx
		pushd /vagrant/
		cp /vagrant/extras/passenger.conf /etc/nginx/passenger.conf
		rm /etc/nginx/nginx.conf
		cp /vagrant/extras/nginx.conf /etc/nginx/nginx.conf
		cp /vagrant/extras/git-lfs-s3-server.conf /etc/nginx/sites-available/git-lfs-s3-server.conf

		mkdir -p /etc/nginx/main.d
		cp /vagrant/extras/secrets-passenger.conf /etc/nginx/main.d/secrets-passenger.conf

		rm -f /etc/nginx/sites-enabled/git-lfs-s3-server.conf
		ln -s /etc/nginx/sites-available/git-lfs-s3-server.conf /etc/nginx/sites-enabled/git-lfs-s3-server.conf

		rm -f /etc/nginx/sites-enabled/default

		cp /vagrant/extras/server.crt /etc/ssl/certs/server.crt
		cp /vagrant/extras/server.key /etc/ssl/private/server.key
		service nginx restart
		popd
		
		
		# Deploy the app to srv/git-lfs-s3
		pushd /vagrant/
		mkdir -p /srv/git-lfs-s3
		cp Gemfile Gemfile.lock config.ru git-lfs-s3-server.rb /srv/git-lfs-s3
		chown -R www-data /srv/git-lfs-s3
		popd
	SHELL
	
	# Run the bundler as non-root as well
	
  
    #Don't run bundler as root
	config.vm.provision "shell", inline: $bundlescript, privileged: false
	
	# Start nginx after everything else
	config.vm.provision "shell", inline: <<-SHELL
		service nginx restart
	SHELL

	
end
