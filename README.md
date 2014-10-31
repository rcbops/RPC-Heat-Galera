# Heat template for deploying MariaDB/Galera with the following:
* 1 salt-master instance
* 1 haproxy instance w/ salt-minion
* 1 MariaDB/Galera bootstrap node with salt-minion
* n MariaDB/Galera Nodes that replicate accross the cluster with salt-minion, where n is an even number.

Still need to work on a healthcheck for MariaDB
Need to clean up inputs to template and add input for choosing flavors.
- Possibly allow choice of flavor for each type of minion

# Overview


This heat template deploys a salt stack on the cloud, then deploys a MariaDB/Galera cluster. It utilizes heat SoftareDeployment, SoftwareCofig and personalities to load pillars, install salt, download the MariaDB/Galera salt states, and run the orchestration runner. 



# Requirements


It requires an image with the heat software config element built into it. You can find the image here:

http://ab031d5abac8641e820c-98e3b8a8801f7f6b990cf4f6480303c9.r33.cf1.rackcdn.com/ubuntu-softwate-config.qcow2

The minimum number of nodes required for this stack is five. Make sure your cloud resources and quota limits can support 5 VMS, 1 Neutron Network, 1 Neutron Router, 1 Nova key pair, and 2 floating ip addresses. Otherwise the stack will fail to build. 



# Parameter descriptions 


The heat template takes in several parameters. Below is the description of each. 

1. keyname: The name of the key that should be used to ssh into the Salt-Master.

2. image: Name of the existing image to use for every vm spun up by this stack. The image should have heat-config and heat-config-script baked in.

3. floating-network-id: UUID of the external network. The private network created by this stack will route to this network. Also, any floating ip's needed by this stack will come this network.

4. coms-key-name: Unique name of the keypair for node to node communications within the stack.

5. minion-count: The number of servers to create in addition to the bootstrap. This number should be even to avoid split-brain. The bootstrap node, in addition to the minion-count, will make the cluster size odd. Make sure to only scale this cluster in odd numbers. 

6. db-username: The username that will be created on the cluser. This will typically be the user required for your app to connect to the app database. 

7. db-user-password: The password for the user above. This is a hidden paramter, meaning it will not be displayed by heat after stack creation. 

8. db-remotehost: The host from which the user will connect. Typically the load balancer of the Galera cluster, but defaults to '%'

9. apps-network: The network UUID of the Apps network. 

10. database: The database that will be created for the app 

11. flavor: The size of the database instances. 
