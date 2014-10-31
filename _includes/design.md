- A separate network and security group is created to house this
database cluster, which lets you isolate your data traffic behind private
networks for security.
- HAProxy will provide load balancing to the database cluster nodes
and is also attached to a Neutron network that your application runs on.
- An optional user and database can be created through the Heat
template.
- Because SaltStack drives the database configuration, scaling the
cluster is as easy as adding additional nodes and setting the appropriate
roles on them.
