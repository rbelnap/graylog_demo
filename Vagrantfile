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

    # Install rsyslog
    yum install -y rsyslog
    systemctl start rsyslog
    systemctl -q enable rsyslog

    # Add rsyslog forwarding option if it does not exist
    if ! grep -q "#{PRIVATE_NET_IP}:1514" /etc/rsyslog.conf; then
      echo "*.* @@#{PRIVATE_NET_IP}:1514;RSYSLOG_SyslogProtocol23Format" >> /etc/rsyslog.conf
      systemctl restart rsyslog
    fi

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
