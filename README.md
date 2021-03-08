
# IPv6 Route Reflector for Docker
When you are using docker with IPv6, you have a private subnet for your containers and then NAT to your host IP (publish ports), however in IPv6 we do not have NAT.
Figure 1 illustrates how the subnet 2002:db8:3:://64 which is used for the container C1 is not visible from the upstream Router1.

`````
                         +--------------------------------------------------------+
                         |                    Docker01.t1.dk                      |
                         |                                                        |
                         |                                                        |
                         |                                                        |
                         |                 +-------------------+                  |
 +---------------+       |                 |Route Table:       |                  |
 |Route Table:   |       |                 |2002:db8:2::/64    |                  |
 |0::/0          |       |                 |2002:db8:3:200::/64|                  |
 |2002:db8:2::/64|       |                 +-------------------+                  |
 +---------------+       |                                                        |
                         |                                                        |
+-----------------+      |    +--------------------+      +----------------+      |
|Router1          |      |    |eth0:               |      |docker0:        |      |
|2002:db8:1::1/64 +-----------+2002:db8:1::100/64  |      |2002:db8:3::1/64|      |
+-----------------+      |    +--------------------+      +-------+--------+      |
                         |                                        |               |
                         |                                        |               |
                         |                               +--------+----------+    |
                         |                               |C1                 |    |
                         |                               |2002:db8:3:200::/64|    |
                         |                               +-------------------+    |
                         |                                                        |
                         |                                                        |
                         |                                                        |
                         |                                                        |
                         |                                                        |
                         +--------------------------------------------------------+


````
Figure1: Hidden Container subnet

Therefore we need a public IPv6 block for our containers, and the Docker host must then proxy Neighbour Discovery Protocol (NDP) Packets or Route the subnet.

Unfortinatly, the NDP proxying is not implemented in the Linux kernel as an experimental feature and the software available is running in userspace mode, which gives a very bad experience. E.g. You loose connectivity for your continers in a few seconds.

To encompass theese problems, you can make the docker host route instead of NDP proxying. This repoistory provides an ansible bundle to do this.

There are two playbooks:
* routereflector-add-host.yml: Configures and update route reflector clients
* routereflector-install-server: Install the BIRD BGP IPv6 router software on route reflectors.

Currently Ubuntu, Centos and RedHat OS are supported.

```
                                 +-----------+
                                 |rr01.t1.dk |
                     +-----------+ASN 65001  +--+
                     |           +-----------+  |           +----------------+
+-----------+        |                          +-----------+docker01.t1.dk  |
| Router1   +--------+                          +-----------+ASN65001        |
| ASN65000  +--------+           +-----------+  |           +----------------+
+-----------+        |           |rr02.t1.dk +--+
                     +-----------+ASN 65001  |
                                 +-----------+
```
Figure: Route Reflector configuration example
