# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/xenial64"

  config.vm.box_check_update = true

  config.vm.hostname = "fashionpeaks.club"

  config.vm.network "private_network", ip: "192.168.56.10"

  config.vm.synced_folder ".", "/opt/workspace"

  config.vm.provision "file", source: "#{ENV['HOME']}/.ssh/id_rsa.pub", destination: "~/.ssh/authorized_keys"

  config.ssh.insert_key = false

  config.ssh.private_key_path = ["#{ENV['HOME']}/.vagrant.d/insecure_private_key", "#{ENV['HOME']}/.ssh/id_rsa"]

  config.vm.provider "virtualbox" do |vb|
    vb.name = "fashionpeaks.club"
    vb.memory = "2048"
    vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    #!/usr/bin/env bash

    apt-get update
    sudo apt-get -y upgrade && sudo apt-get autoremove -y

    # Install Jdk if javac not exist.
    if ! [ -x "$(command -v javac)" ]; then
      sudo add-apt-repository -y ppa:webupd8team/java
      sudo apt-get update
      echo debconf shared/accepted-oracle-license-v1-1 select true | sudo debconf-set-selections 
      echo debconf shared/accepted-oracle-license-v1-1 seen true | sudo debconf-set-selections
      sudo apt-get -y install oracle-java8-installer
    fi

    # Install maven if mvn command not exist.
    if ! [ -x "$(command -v mvn)" ]; then
      apt-get install maven -y
    fi

    http_proxy_val=#{ENV['VAGRANT_HTTP_PROXY']}

    if [[ -f "/opt/workspace/settings.xml" && ! -z $http_proxy_val ]]; then
        echo "settings.xml file found."
        sudo mkdir -p ~/.m2
        sudo cp -f /opt/workspace/settings.xml ~/.m2/
    else
      echo "Removing settings.xml"
      sudo rm -rf ~/.m2/settings.xml
    fi

    # Install MongoDB if mongod not exist.
    if ! [ -x "$(command -v mongod)" ]; then
      sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
      echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
      sudo apt-get update
      sudo apt-get install -y mongodb-org
    fi

    sudo service mongod restart

    # Install NodeJs if node not exist.
    if ! [ -x "$(command -v node)" ]; then
      sudo apt-get install curl python-software-properties gcc g++ make -y
      curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
      sudo apt-get install nodejs -y
      curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
      echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
      sudo apt-get update && sudo apt-get install yarn -y
    fi

    # Install Nginx if nginx not exist.
    if ! [ -x "$(command -v nginx)" ]; then
      sudo apt-get install nginx -y
    fi

    sudo systemctl restart nginx

    mkdir -p /opt/app/bin /opt/workspace/code-repos

    cd /opt/workspace/code-repos

    # Clone the entheos-api repo if entheos-api directory is not exist.
    if [ ! -d entheos-api ]; then
        git clone https://github.com/CrazyEntheos/entheos-api.git
    fi

    # Build the Api Package
    pushd entheos-api
      git clean -df
      git pull
      mvn -U clean install
      sudo cp target/store-api-*.jar /opt/app/bin/store-api.jar
    popd

    cp /opt/workspace/conf/store-api.service /etc/systemd/system/store-api.service

    systemctl enable store-api && systemctl daemon-reload && systemctl restart store-api

    # Clone the entheos-ui repo if entheos-ui directory is not exist.
    if [ ! -d entheos-ui ]; then
        git clone https://github.com/CrazyEntheos/entheos-ui.git
    fi

    # Build the UI dist
    pushd entheos-ui
      git clean -df
      git pull
      echo "Install npm packages"
      npm install --no-bin-links
      npm install -g @angular/cli@6.0.0
      echo "Executing ng build"
      ng build --prod

      if [[ -f "/opt/workspace/ssl-certs/fashionpeaks.crt" && -f "/opt/workspace/ssl-certs/fashionpeaks.key"  ]]; then
        echo "ssl certificates exist and configuring application over https"
        sudo mkdir -p /opt/ssl
        sudo cp -f /opt/workspace/ssl-certs/* /opt/ssl
        sudo cp /opt/workspace/conf/https-fashionpeaks.conf /etc/nginx/sites-available/fashionpeaks.conf
      else
       echo "application configured over http"
       sudo cp /opt/workspace/conf/fashionpeaks.conf /etc/nginx/sites-available/fashionpeaks.conf
      fi

      sudo chmod 644 /etc/nginx/sites-available/fashionpeaks.conf
      sudo ln -f -s /etc/nginx/sites-available/fashionpeaks.conf /etc/nginx/sites-enabled/
      sudo mkdir -p /var/www/fashionpeaks
      sudo rm -rf /opt/workspace/code-repos/entheos-ui/dist/fashionpeaks
      sudo mv /opt/workspace/code-repos/entheos-ui/dist/my-first-app /opt/workspace/code-repos/entheos-ui/dist/fashionpeaks
      rsync -a  /opt/workspace/code-repos/entheos-ui/dist/fashionpeaks /var/www

      sudo service nginx restart
    popd
  SHELL
end
