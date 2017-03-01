# RabbitMQ Set Up
This document has details regarding setting up RabbitMQ clustered environment

# Installation
  - echo 'deb http://www.rabbitmq.com/debian/ testing main' |
        sudo tee /etc/apt/sources.list.d/rabbitmq.list
  - wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc |
        sudo apt-key add -
  - sudo apt-get update
  - sudo RUNLEVEL=1 apt-get -y install rabbitmq-server

# Ports to be Opened
  - 4369 (epmd)
  - 5672, 5671 (AMQP 0-9-1 and 1.0 without and with TLS)
  - 25672, 25671. This port used by Erlang distribution for inter-node and CLI tools communication and is allocated from a dynamic range.
  - 15672 for management plugin

# Increase File Descriptors
Uncomment and edit following line in file `/etc/default/rabbitmq-server` : 
  - ulimit -n 65536
  
# Set up Cluster
  - Please make sure each VM in cluster resolves to FQDN when typed `hostname -f`. Run following command to set desired FQDN on a VM `sudo hostnamectl set-hostname FQDN` 
  - Execute installation steps and open mentioned ports on all VMs that needs to be included in cluster
  - Create file `/etc/rabbitmq/rabbitmq-env.conf` with content `RABBITMQ_USE_LONGNAME=true`
  - Start rabbitmq server once using command `sudo service rabbitmq-server start`. This will create few default folders. Most prominent of them are:
    - `/var/lib/rabbitmq/` . Rabbitmq stores persistent data here. Please add virtual link to larger volutme on your VM to this folder if expected static data is higher than default VM hard disk limit of 20 GB.  
    - `/var/log/rabbitmq/`. Rabbitmq creates different log files here. Good folder to check into in case of any exception/error
  - Stop rabbitmq by executing following commands in sequence:
    - `sudo rabbitmqctl stop_app` 
    - `sudo rabbitmqctl stop`
  - Give write permission to erlang cookie file using `sudo chmod u+rw /var/lib/rabbitmq/.erlang.cookie`
  - Make sure content of erlang.cookie file is same for all the VMs in a cluster
  - Exectute following steps on all VMs in cluster except one reference node:
    - Start rabbitmq server using  `sudo rabbitmq-server -detached`
    - Stop rabbitmq app using `sudo rabbitmqctl stop_app`
    - Join the cluster by running `sudo rabbitmqctl join_cluster rabbit@cluster_name` where cluster_name is FQDN of reference node.
    - Start rabbitmq app using `sudo rabbitmqctl start_app`
  - You need not create user separately for different VMs in cluster, User created on one VM will be available across cluster. To enable UI portal for a particular VM, you need to execute Enable UI Portal step mentioned above on each VM. 

# Enable UI Portal
  ```sh
  $ sudo rabbitmq-plugins enable rabbitmq_management
  ```
# Set up user for remote access
  ```sh
    $ sudo rabbitmqctl add_user <username> <password>
    $ sudo rabbitmqctl set_user_tags <username> <role> 
    $ sudo rabbitmqctl set_permissions -p / <username> ".*" ".*" ".*" 
  ```
# Set up Queue duplication policy
 ```sh
 $ rabbitmqctl set_policy ha-ots "^ots" \
   '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}'
  ```
# Network Partition
Here is details of network partition: http://www.rabbitmq.com/partitions.html
  