- A separate network and security group is created to house this
database cluster, which lets you isolate your data traffic behind private
networks.
- HAProxy provides load balancing to the database cluster nodes
and is also attached to a Networking (neutron) network where your application runs.
- An optional user and database can be created through the Orchestration (heat)
template.
- Because SaltStack drives the database configuration, scaling the
cluster is as easy as adding additional nodes with the appropriate
roles.
