# Heat template for deploying MariaDB/Galera with the following:
* 1 salt-master instance
* 1 haproxy instance w/ salt-minion
* 1 MariaDB/Galera bootstrap node with salt-minion
* 2 MariaDB/Galera Nodes in addition to the bootstrap node with salt-minion on them. 

# Description:


This heat template deploys a salt stack on the cloud, then deploys a MariaDB/Galera cluster. It utilizes heat SoftareDeployment, SoftwareCofig and personalities to load pillars, install salt, download the MariaDB/Galera salt states, and run the orchestration runner. 

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

* An Ubuntu image (12.04 or newer) preconfigured with heat-cfntools and heat confog-script. 
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
heat stack-create galera-stack -f galera-stack.yaml \
  -e env.yaml -P flavor=m1.small;floating-network-id=<NET_ID>; \
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
