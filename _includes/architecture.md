Traditional relational databases are primarily assumed to scale up,
not out. This means that running relational databases on a Cloud platform
can introduce new complexities. This template deploys three
MariaDB/Galera nodes and an HAProxy Load Balancer, and is meant to be used as 
a MySQL compatible database of your application. You can easily attach this 
template as a MySQL provider for your own Orchestration (heat) application 
templates, or you can attach this cluster to an existing application on an 
OpenStack Cloud using Networking (neutron).
