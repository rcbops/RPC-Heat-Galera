# Heat template for deploying MariaDB/Galera with the following:
* 1 salt-master instance
* 1 haproxy instance w/ salt-minion
* 1 MariaDB/Galera bootstrap node with salt-minion
* 2 MariaDB/Galera Nodes in addition to the bootstrap node with salt-minion on them. 

# Description:


This heat template deploys a salt stack on the cloud, then deploys a MariaDB/Galera cluster. It utilizes heat SoftwareDeployment, SoftwareConfig and personalities to load pillars, install salt, download the MariaDB/Galera salt states, and run the orchestration runner. 

Requirements
============
* A Heat provider that supports the following:
  * OS::Neutron::Net
  * OS::Neutron::Subnet
  * OS::Neutron::Router
  * OS::Neutron::RouterInterface
  * OS::Neutron::FloatingIP
  * OS::Neutron::FloatingIPAssociation
  * OS::Neutron::Port
  * OS::Heat::SoftwareConfig
  * OS::Heat::SoftwareDeployment
  * OS::Heat::RandomString
  * OS::Heat::ResourceGroup
  * OS::Nova::Server
  * OS::Nova::KeyPair

* An Ubuntu image (12.04 or newer) preconfigured with heat-cfntools and heat config-script. 
Instructions for creating a heat-cfntools enabled image for use with Heat can be 
found [here] (http://docs.openstack.org/developer/heat/getting_started/jeos_building.html).

* An OpenStack username, password, and tenant id.
* [python-heatclient](https://github.com/openstack/python-heatclient)
`>= v0.2.12`:

Heat-client Usage
=============
Here is an example of how to deploy this template using the
[python-heatclient](https://github.com/openstack/python-heatclient):

```
heat stack-create galera-stack -f mariadb_stack.yaml \
  -P flavor=m1.small;floating-network-id=<NET_ID>; \
  keyname=<KEYNAME>;image=<IMAGE_ID>;db-username=<username>;db-user-password=<password>; \
  db-remotehost=<host>;apps-network=<APP_NET_ID>;database=<db_name>
```

# Parameter descriptions 


The heat template takes in several parameters. Below is the description of each. 

1. keyname: The name of the key that should be used to ssh into the Salt-Master.

2. image: Name of the existing image to use for every vm spun up by this stack. The image should have heat-config and heat-config-script baked in.

3. floating-network-id: UUID of the external network. The private network created by this stack will route to this network. Also, any floating ip's needed by this stack will come this network.

4. db-username: The username that will be created on the cluser. This will typically be the user required for your app to connect to the app database. 

5. db-user-password: The password for the user above. This is a hidden paramter, meaning it will not be displayed by heat after stack creation. 

6. db-remotehost: The host from which the user will connect. Typically the load balancer of the Galera cluster, but defaults to '%'

7. apps-network: The network UUID of the Apps network. 

8. database: The database that will be created for the app 

9. flavor: The size of the database instances.

#Architecture
This database was intended to be ran in conjunction with specific applications. However, it can be configured manually for any application. Databases and tables can be added by logging in as root. See the "Accessing mysql cluster as root" section. 

The diagram below illustrates the architecture
![](http://718016a9d23737f3d804-7671e86526a10735410d8ae5040e7d55.r41.cf1.rackcdn.com/GaleraHeatSolution.png)

The application network and the neutron external network are required to be in place before the stack is created. The template will ask for these networks as parameters (See above section on heat parameters).

The template creates an HAProxy node which load balances between the database nodes. This node is both in the application and database network. 

The Salt-master node is the only node in this stack with a floating-ip. It's responsible for the configuration management of the database cluster. Furthermore, it can be used to access the other nodes in the database network, including the HAProxy node.

The database nodes will be on their own network, accessible by an ssh key "/root/.ssh/coms_rsa" placed on all the nodes the template creates.


#Quick Start Guide
##Running the heat templates. 
With the rackspace solution tab installed, navigate to the Rackspace tab on the left, then to the Solutions section. A screen like the one below should appear.
![](http://718016a9d23737f3d804-7671e86526a10735410d8ae5040e7d55.r41.cf1.rackcdn.com/SolutionsTab.png)
Select the MariaDB Galera Cluster solution. A prompt will appear asking for some paramters.

Choose the openstack resources that fit your environment from the drop down menu. Then, choose a username, password, remote host, and database to be configured on the MariaDB/Galera cluster.
##Accessing the application database as the application user.
Logging into the database as the app user requires the username and password specified in the paramters. Note that the database will only be accessible from the remote host specified on creation.

In this deployment, the HAproxy node can be used to access the database. Since the salt-master node is the only node with a floating IP, it is only possible to access the HAProxy node from the salt-master node. Simply ssh into the salt-master node, then, by using the SSH key set up on stack creation, ssh into the HAProxy node. 

SSH'ing into the salt-master
```shell
root@yourbox:~# ssh -i <keypair> ec2-user@<salt-master-ip>
```
SSH'ing into the HAProxy node. Make sure to use the IP address on the same network as the salt-master.
```shell
$ sudo su
root@salt-master:~# ssh -i /root/.ssh/coms_rsa root@<haproxy-ip>
```
From the haproxy node, the database created for the app can be accessed. 
```shell
root@haproxy-node:~# mysql -u<user-specified> -p<password-specified> -h 127.0.0.1
```
##Accessing mysql cluster as root
If more databases, tables, or users need to be created, the following method can be used to log in as root user. Note that by default, remote login as root has been disabled. 

First, log into one of the database nodes from the salt-master. 
```shell
root@salt-master:~# ssh -i /root/.ssh/coms_rsa root@<database-node-ip>
```
To obtain the root password for your database, navigate the stack's resources tab, then find the admin_password resource. The value for this resource is the root password for your database. 
![](http://718016a9d23737f3d804-7671e86526a10735410d8ae5040e7d55.r41.cf1.rackcdn.com/admin_password.png)

Use the admin_password resource to login to the database as root.
```shell
root@database-node:~# mysql -uroot -p<admin-password>
```


