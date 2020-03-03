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
    rpm --import docker-key \
      /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 \
      http://download.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7

    # Install Docker Community Edition
    yum-config-manager --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
    yum install -y docker-ce docker-ce-cli containerd.io
    systemctl start docker
    systemctl -q enable docker
    usermod -aG docker vagrant

    # Convenience
    yum install -y vim

    # Install jq
    yum install -y epel-release
    yum install -y jq

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

  # Start compose services and add default input
  config.vm.provision "shell", inline: <<-SHELL

    # Remove old keys and create directories
    mkdir -p /vagrant/pki
    rm -r /vagrant/pki/*
    mkdir -p /vagrant/pki/{fluentd,graylog}

    # Generate and install TLS keys
    cd /vagrant/pki

    # Generate Graylog's CA
    openssl genrsa -out rootCA.key 4096 2> /dev/null
    openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 \
        -out rootCA.crt -subj "/C=US/ST=GA/O=MyOrg/CN=localhost" 2> /dev/null

    # Generate Fluentd's keys
    openssl genrsa -out fluentd.key 4096 2> /dev/null
    openssl req -new -sha256 -key fluentd.key \
        -subj "/C=US/ST=GA/O=MyOrg/CN=localhost" -out fluentd.csr 2> /dev/null

    # Sign Fluentd's certificate
    openssl x509 -req -in fluentd.csr -CA rootCA.crt -CAkey rootCA.key \
        -CAcreateserial -out fluentd-signed.crt -days 500 -sha256 2> /dev/null

    mv fluentd*.* fluentd/
    mv root*.* graylog/

    # Bring up containers
    cd /vagrant
    /usr/local/bin/docker-compose up -d 2> /dev/null
    cd /vagrant/wordpress
    /usr/local/bin/docker-compose up -d 2> /dev/null
    cd /vagrant

    # Wait 120 seconds for Graylog to come online
    INSTALL_INPUT=0
    SECONDS=0
    while true
    do
      GRAYLOG_STATE=$(
        docker inspect vagrant_graylog_1 \
          | jq --raw-output '.[] | .State.Health.Status')

      if [[ "$GRAYLOG_STATE" == "healthy" ]]; then
        echo "Graylog is available."
        INSTALL_INPUT=1
        sleep 5
        break
      elif [[ "$GRAYLOG_STATE" != "starting" ]]; then
        echo "Something is wrong with Graylog. Aborting."
        break
      elif [[ $SECONDS -le 120 ]]; then
        echo "Waiting for Graylog ($SECONDS/120 seconds)"
        sleep 10
      else
        echo "Waiting on Graylog timed out. Aborting."
        break
      fi
    done

    # Check for existing GELF TCP Input
    INPUTSTATE=$(
      curl -s -X GET \
          -H "Content-Type: application/json" \
          -H "X-Requested-By: cli" \
          -u admin:admin \
          "http://graylog.172.28.128.30.xip.io:8080/api/system/inputstates")

    INPUT_TYPES=$(echo $INPUTSTATE | jq --raw-output '.states | .[] | .message_input.type')

    for TYPE in $INPUT_TYPES; do
      if [[ "$TYPE" == "org.graylog2.inputs.gelf.tcp.GELFTCPInput" ]]; then
        echo "Found GELF TCP input in Graylog, aborting input installation."
        INPUT_INSTALL=1
        break
      fi
    done

    # Install GELF TCP Input
    if [[ $INSTALL_INPUT -eq 1 ]]; then
      echo "Installing GELF TCP input"
      curl -i -s -X POST \
          -H "Content-Type: application/json" \
          -H "X-Requested-By: cli" \
          -u admin:admin \
          "http://graylog.172.28.128.30.xip.io:8080/api/system/inputs" \
          -d @GELFTCPInput.json
    fi

  SHELL

end
