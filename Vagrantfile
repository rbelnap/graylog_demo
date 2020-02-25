# vi: set ft=ruby :

PRIVATE_NET_IP = "172.28.128.30"

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.network "private_network", ip: PRIVATE_NET_IP
  config.vm.synced_folder ".", "/vagrant", type: "nfs"

  config.vm.provider "virtualbox" do |vbox|
    vbox.memory = 4096
    vbox.cpus = 4
  end

  config.vm.provision "shell", inline: <<-SHELL

    # Set SELinux to permissive
    setenforce 0
    sed -i "s/SELINUX=enforcing/SELINUX=permissive/g" /etc/selinux/config

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

    # Convenience
    yum install -y vim

    # Install apache
    yum install -y httpd
    systemctl start httpd
    systemctl -q enable httpd

    # Install rsyslog
    yum install -y rsyslog
    systemctl start rsyslog
    systemctl -q enable rsyslog

    # Add rsyslog forwarding option if it does not exist
    if ! grep -q "127.0.0.1:5140" /etc/rsyslog.conf; then
      echo "*.* @127.0.0.1:5140" >> /etc/rsyslog.conf
      systemctl restart rsyslog
    fi

  SHELL

  # Install newest docker-compose
  config.vm.provision "shell", path: "install-compose.sh"

  # Start compose services
  config.vm.provision "shell", inline: <<-SHELL
    cd /vagrant
    /usr/local/bin/docker-compose up -d 2> /dev/null
    cd /vagrant/wordpress
    /usr/local/bin/docker-compose up -d 2> /dev/null
    echo "Waiting 60 seconds for Graylog to become available..."
    sleep 60
    cd /vagrant
    curl -i -X POST \
        -H "Content-Type: application/json" \
        -H "X-Requested-By: cli" \
        -u admin:admin \
        "http://graylog.172.28.128.30.xip.io:8080/api/system/inputs" \
        -d @GELFUDPInput.json
  SHELL

end
