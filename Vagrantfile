# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.provision "shell", inline: <<-SHELL

    # Import GPG keys
    curl -s https://download.docker.com/linux/centos/gpg -o docker-key
    rpm --import docker-key /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

    # Install Docker Community Edition
    yum-config-manager --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
    yum install -y docker-ce docker-ce-cli containerd.io
    systemctl start docker
    usermod -aG docker vagrant

  SHELL

end
