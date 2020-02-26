# Graylog Demo


This is a demonstration of Graylog, a centralized log management system featuring a shell provisioned CentOS 7 Vagrant box. To illustrate various log collection methods `httpd`, `rsyslog` and `docker` are installed and a simple WordPress instance is deployed via Docker Compose. Log collection incorporates Fluentd to ship logs into a Graylog instance from containers, the syslog, and arbitrary filesystem logs.

This demonstration assumes you are familiar with using Vagrant + VirtualBox to automate the installation of virtual machines, although you can reference the Vagrantfile's shell provisioning sections to manually set up a system if you so desire. Please install these prerequisites before attempting the quick start below.

#### Notes about setup
- This demonstration uses Traefik for routing and the [xip.io](http://xip.io/) wildcard DNS service. If DNS fails to resolve you may want to set the domains to the IP inside your operating system's hosts file.

- Vagrant will provision a virtual machine with a static private Class B address (specifically `172.28.128.30`). If you would like to change this IP address to something different you will need to change the `PRIVATE_NET_IP` variable and the scripted API calls in the `Vagrantfile`. You'll also need to modify the few wildcard DNS references to it in the two `docker-compose.yml` files.

- Vagrant is set to allocate 4 cores and 4 GB of RAM, you may need to adjust this for your machine if necessary.

- After deploying, Graylog takes the longest to become available and it may take 30 seconds to a few minutes to bring it up depending on your machine.


#### _This project is a demonstration only and should not be used in a production environment._


## Quick Start
_This section assumes you will be using the default `172.28.128.30` IP address_
1. Clone the repository and navigate inside its directory
2. Create and provision the VM using `vagrant up`
3. Navigate to [http://graylog.172.28.128.30.xip.io:8080/](http://graylog.172.28.128.30.xip.io:8080/)
4. Login using `admin` for both the username and password.
5. Click on `Search` on the top menu on the left
6. You may want to "Search in all messages" on the left under the top menu
7. Press the start button on the top right to start updating the feed every second

#### Docker Test
- Generate Docker logs by simply navigating to the WordPress install page [http://wordpress.172.28.128.30.xip.io:8080/](http://wordpress.172.28.128.30.xip.io:8080/)

#### File Test
- Collect logs from Apache's `access_log` file by going to [http://172.28.128.30/](http://172.28.128.30/)

#### Syslog Test
1. Go back to the terminal inside the project's directory and type `vagrant ssh`
2. You can test Syslog collection with `logger` e.g. `logger -t test Hello world` (or just wait for some to appear)

### Copyrights and Licenses
Copyright (C) 2020  Kris Lamoureux

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, version 3 of the License.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.
