NATApp User Guide
=================

The NATApp User Guide contains information about configuration,
administration, management, using and troubleshooting the feature.

Overview
--------

NATApp provides network different types of address translation
functionality for OpenDaylight. After installing this feature, network
administrators can select the type of NAT functionality they want to
enable by sending a REST API command. Subsequently, the user may enter
the gloabl IP addresses to the YANG Data Store through REST APIs. When
an OpenDaylight managed enterprise network with local IPs tries to
connect to external networks such as Internet, NATApp comes into play
and installs appropriate flow rules at the OpenFlow switch for
bidirectional NAT translation.

NATApp Architecture
-------------------

NATApp listens on OpenFlow southbound interface for Packet\_In messages.
The application parses the message for header information. If the
received message has a local IP address the application installs rules
on the OpenFlow switch for network address translation from local to
global IP addresses. NATApp has NATPacketHandler class that implements
the PacketProcessing interface to override the OnPacketReceived
notification by which the application is notified of Packet\_In
messages.

Configuring NATApp
------------------

REST APIs are available at the following URI:
http://localhost:8181/apidoc/explorer/index.html#!/natapp(2016-01-25)

Mininet Topology
~~~~~~~~~~~~~~~~

::

    sudo mn --mac --topo=single,10 --controller=remote,ip=127.0.0.1,port=6653

Install a flow to flood the ARP packets.

::

    sh ovs-ofctl add-flow s1 dl_type=0x0806,actions=FLOOD

Check the flow for ARP Flooding

::

    sh ovs-ofctl dump-flows s1

Administering or Managing NATApp
--------------------------------

Static NAT and Dynamic NAT
~~~~~~~~~~~~~~~~~~~~~~~~~~

First user has to select the type of NAT he wants by using the following
URI:

POST URI
    http://localhost:8181/restconf/operations/natapp:nat-type

Sample Input
    {"natapp:input": { "type:static":""}}

Sample Input
    {"natapp:input": { "type:dynamic":""}}

Then user can inject the Global IPs using the following URI

PUT URI
    http://localhost:8181/restconf/config/natapp:staticNat/

Sample Input
    {"natapp:staticNat": {"globalIP":["172.0.0.1/32","172.0.0.2/32",
    "172.0.0.3/32", "172.0.0.4/32", "172.0.0.5/32", "172.0.0.6/32",
    "172.0.0.7/32", "172.0.0.8/32", "172.0.0.9/32", "172.0.0.10/32"] }}

From mininet verify any pair of hosts can ping each other. The NATApp
modifies the destination IP address of the ICMP Echo request with the
global IP address. Check the mininet flows for this modification.

::

    sh ovs-ofctl dump-flows s1

Port Address Translation (PAT)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

User can select PAT by using the following URI.

POST URI
    http://localhost:8181/restconf/operations/natapp:nat-type

Sample Input
    {"natapp:input": { "type:pat":""}}

Then user can inject the Global IPs using the following URI

PUT URI
    http://localhost:8181/restconf/config/natapp:patNat/

Sample Input
    {"natapp:patNat": {"globalIP":"172.0.0.1/32"}}

From Mininet use the command as xterm h1 h5. At h5 give the following
commands

::

    $ ip r add 172.0.0.1/32 dev h5-eth0
    $ arp -s 172.0.0.1 00:00:00:00:00:01
    $ nc -l 5000

At h1, Give the following command

::

    $ echo "TCS" | nc -p 8000 10.0.0.5 5000

::

    mininet> sh ovs-ofctl dump-flows s1
    NXST_FLOW reply (xid=0x4):
     cookie=0x0, duration=811.272s, table=0, n_packets=5, n_bytes=342, idle_age=13, priority=210,tcp,in_port=1,tp_src=8000 actions=mod_nw_src:172.0.0.1,mod_tp_src:2000,output:5
     cookie=0x0, duration=499.843s, table=0, n_packets=2, n_bytes=84, idle_age=13, arp actions=FLOOD
     cookie=0x0, duration=811.203s, table=0, n_packets=3, n_bytes=206, idle_age=13, priority=209,tcp,in_port=5,tp_dst=2000 actions=mod_nw_dst:10.0.0.1,mod_tp_dst:8000,output:1

