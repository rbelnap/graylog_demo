# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.network "private_network", ip: "172.28.128.30"
  config.vm.synced_folder ".", "/vagrant", type: "nfs"

  config.vm.provider "virtualbox" do |vbox|
    vbox.memory = 4096
    vbox.cpus = 4
  end

  config.vm.provision "shell", inline: <<-SHELL

    # Import GPG keys
    curl -s https://download.docker.com/linux/centos/gpg -o docker-key
    rpm --import docker-key /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

    # Install Docker Community Edition
    yum-config-manager --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
    yum install -y docker-ce docker-ce-cli containerd.io
    systemctl start docker
    systemctl -q enable docker
    usermod -aG docker vagrant

  SHELL

  # Install newest docker-compose
  config.vm.provision "shell", path: "install-compose.sh"

  # Start compose services
  config.vm.provision "shell", inline: <<-SHELL
    cd /vagrant
    /usr/local/bin/docker-compose -f wordpress.yml up -d 2> /dev/null
    /usr/local/bin/docker-compose -f graylog.yml up -d 2> /dev/null
  SHELL

end
