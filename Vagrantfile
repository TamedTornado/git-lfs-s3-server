# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

	 
	config.vm.provision "shell", inline: <<-SHELL
	  apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
	  apt-get install -y apt-transport-https ca-certificates
	  sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger xenial main > /etc/apt/sources.list.d/passenger.list'
	   apt-get update
	   apt-get install -y nginx git nginx-extras passenger
	  gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
	  curl -sSL https://get.rvm.io | bash -s stable
	  source /etc/profile.d/rvm.sh
	   rvm install ruby-2.2.3
	   rvm use ruby-2.2.3
	   cd /vagrant/
	   gem install bundler
	   bundle
	   cp /vagrant/extras/passenger.conf /etc/nginx/passenger.conf
	   rm /etc/nginx/nginx.conf
	   cp /vagrant/extras/nginx.conf /etc/nginx/nginx.conf
	   cp /vagrant/extras/git-lfs-s3-server.conf /etc/nginx/sites-available/git-lfs-s3-server.conf

	   mkdir -p /etc/nginx/main.d
	   cp /vagrant/extras/secrets-passenger.conf /etc/nginx/main.d/secrets-passenger.conf

	   rm /etc/nginx/sites-enabled/git-lfs-s3-server.conf
	   ln -s /etc/nginx/sites-available/git-lfs-s3-server.conf /etc/nginx/sites-enabled/git-lfs-s3-server.conf

	   rm -f /etc/nginx/sites-enabled/default
	   
	   cp /vagrant/extras/server.crt /etc/ssl/certs/server.crt
	   cp /vagrant/extras/server.key /etc/ssl/private/server.key
	   service nginx restart
	   SHELL
end
