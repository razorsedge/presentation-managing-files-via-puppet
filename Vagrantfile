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
  config.vm.box = "puppetlabs/centos-7.0-64-puppet"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080
#  config.vm.network "forwarded_port", guest: 8000, host: 8000

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
  config.vm.synced_folder "modules", "/etc/puppet/modules"

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

  config.ssh.forward_x11 = true

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
  config.vm.provision "shell", inline: <<-SHELL
    service nfslock stop
    service rpcbind stop
    service auditd stop
    service abrt-ccpp stop
    service abrtd stop
    service iprdump stop
    service iprinit stop
    service iprupdate stop
    service avahi-daemon stop
    service postfix stop
    cat <<EOF1 >/etc/yum.repos.d/google-chrome.repo
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/\\\$basearch
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
EOF1
    yum -y -d1 -e1 install yum-presto deltarpm epel-release xorg-x11-xauth

    yum -y -d1 -e1 install git nodejs npm shellinabox google-chrome-stable xorg-x11-xauth
    git clone https://github.com/hakimel/reveal.js.git /opt/presentation
    cd /opt/presentation
    npm install --quiet -g grunt-cli
    npm install --quiet
    #sed -i "s|port: port|port: port,\\n\\t\\t\\t\\t\\thostname: \'\'|g" /opt/presentation/Gruntfile.js
    git clone https://github.com/krishnasrinivas/wetty /opt/wetty
    cd /opt/wetty
    npm install --quiet
    sed -i -e '/ pty.spawn/s|ssh|/bin/bash|' -e '/ pty.spawn/s|\\[ssh.*\\],|\\[\\],|' /opt/wetty/app.js

    yum -y -d1 -e1 install git nodejs rubygem-bundler ruby-devel sqlite-devel
    git clone https://github.com/ConradIrwin/showterm.io.git /opt/showterm
    cd /opt/showterm
    sed -i -e '/^ruby/ s|2.1.1|2.0.0|' -e "s|gem 'pg'|gem 'sqlite3'|" /opt/showterm/Gemfile
    bundle install --path=/opt/showterm
    cat <<EOF2 >/opt/showterm/config/database.yml
production:
  adapter: sqlite3
  database: /opt/showterm/db/showterm.sqlite3
  pool: 5
  timeout: 5000
development:
  adapter: sqlite3
  database: /opt/showterm/db/showterm.sqlite3
  pool: 5
  timeout: 5000
EOF2
    if [ -f /vagrant/presentation/showterm.sqlite3 ]; then
      cp -p /vagrant/presentation/showterm.sqlite3 /opt/showterm/db/showterm.sqlite3
    else
      bundle exec rake db:create db:migrate db:seed
    fi
    nohup bin/rails server --port=3001 >/nohup.rails.out &
    sed -i -e '/^div.terminal/a\
    zoom: 150%;' /opt/showterm/app/assets/stylesheets/scripts.css.scss

    gem install showterm
    cat <<EOF3 >/etc/profile.d/showterm.sh
export SHOWTERM_SERVER=http://localhost:3001/
EOF3
    chmod +x /etc/profile.d/showterm.sh

    /bin/cp -a /vagrant/presentation/* /opt/presentation/
    touch /etc/puppet/hiera.yaml
#    puppet config set basemodulepath /vagrant/modules:/etc/puppet/modules:/usr/share/puppet/modules
    puppet config set disable_warnings deprecations
#    su - vagrant -c "puppet apply --noop /vagrant/modules/example/tests/A.pp;puppet config set basemodulepath /vagrant/modules:/home/vagrant/.puppet/modules:/usr/share/puppet/modules"
    su - vagrant -c "puppet apply --noop /vagrant/modules/example/tests/A.pp;puppet config set basemodulepath /home/vagrant/.puppet/modules:/etc/puppet/modules:/usr/share/puppet/modules"
    echo vagrant | passwd --stdin vagrant

    #systemctl stop firewalld
    #firewall-cmd --zone=public --add-port=8000/tcp --permanent
    #firewall-cmd --zone=public --add-port=3000/tcp --permanent
    #firewall-cmd --zone=public --add-port=3001/tcp --permanent
    #firewall-cmd --reload
  SHELL
end
