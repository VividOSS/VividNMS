# VividNMS
Yet another Network Monitoring System

It's just a wild idea at the moment, but I want to re-imagine the whole NMS thing from the ground up, using existing tools, ideas, and code where possible.

## Where I'm at

Let's face it, there is no such thing as a perfect monitoring system.  Each was created with specific goals and purposes.  No one solution works for all organizations and networks.  Without trying to get too deep into the minds of the authors, here are some pros and cons of each from the perspective of a systems and network admin with over 20 years experience building and operating service provider networks.

Every platform  worth consideration comes with a steep learning curve.  Those that are too easy are almost always too simplistic to meet the needs of a large network that's expected to perform with 4 or 5 9's of reliability.

### Nagios (XI)

I've been through quite a few systems in my time, but keep coming back to Nagios, even back when it was still NetSaint.  However, Nagios does have some serious draw backs, namely it's inability to collect and manage times series data, lack of a usable admin UI, and even it's 20-year love affair with cgi-bin and basic http authentication.

Nagios XI attempts to resolve some of these issues, but falls short of meeting needs for large environments.  They added some wizards, which I would imagine work well for small environments, but are quite clunky for larger environments.  The only wizards I've found to be even remotely useful, are those for adding routers and switches.  They really take the pain out of adding things to MRTG.  However, that's where it ends, for both wizards and MRTG.  However, that's the beginning and end of both the Wizards and the MRTG config.  While Nagios XI can add stuff to the MRTG configuration, it is not able to manage the MRTG configuration.  While the Wizards can quickly add hosts and even services to monitoring, they do not work well with a large, template driven environment.

XI's other shining benefit, is the Core Config Manager.  Based on the NagiosQL project, the Nagios team did a pretty awesome job extending and improving it.  However, at it's core, it's still NagiosQL.

For me, the best features for Nagios are:

 - Template driven configuration
 - Easy parent/child host dependencies
 - Huge universe of plugins to monitor damned near anything you want
 - XI offers a very usable interface, but at a cost
 - XI's unlimited services makes it nice for corporate networks

Cons:
- No native configuration UI
- No native support for time series data collection and reporting
- Inability to handle circular dependencies
- XI is expensive for provider networks (tons of devices, few services)

### Zabbix

I discovered Zabbix several years ago.  It took a few tries to get over the initial learning curve and actually get it to do some useful work.  The host and low level discovery is pretty awesome, but not well suited to large environments.

The learning curve to create the templates and prototypes for data collection, monitoring, alerting, graphing, etc... was pretty steep, but well worth the effort.  It really makes you think about what you're doing.  Getting email, SMS, and even Slack notifications was fairly trivial.

I currently use Zabbix to collect SNMP data from routers, switches, and access points, which works a serious treat with Grafana on the frontend to build and display graphs.

There are a few things that keep me from replacing Nagios with Zabbix though, and that is the lack of simple parent/child relationships.  Yes, I know host dependencies are possible, but the extra effort to navigate 1000's of devices to define all those relationships is not worth the effort if it can't be done in the same place and time the hosts are configured.

Pros:

 - Discovery is awesome
 - SNMP time series data collection is awesome
 - Templates & Prototypes are awesome

Cons

 - Lack of simple parent/child relationships
 - Discovery on large networks is resource intensive and time consuming to test and tune.

### LibreNMS

To be honest, I've not given Libre a proper chance to prove itself.  From what I remember, the discovery process was pretty awesome.  It put Zabbix to shame.  IIRC, it went so far as to look at everything in SNMP to discover new things.  Point it at your router, and it'll crawl your entire network and discovery anything and everything it can.  Needless to say, it's very resource intensive.  I had to terminate my testing because the storage array in my lab server was getting trashed.  At some point, I want to toss it on a box with sufficient resources to do a proper test.

Pros:

 - Recursive discovery

Cons: 

 - Steep resource requirements, even for testing a small portion of one's network
 - Lack of simple parent/child relationships (based on long standing feature requests)

### OpenNMS

I'm not sure why I'm even including this in my list.  I'm aware of it.  I've looked through the documentation.  I've even installed it once.  If I'm being honest though, I didn't even take the time to monitor even a few devices.

OpenNMS looks to be a very complete tool, suitable for either a very small network being run by a single person who likes to play, or on a very large network with a large operations team with the time and resources to get it done.

### Others

I'm aware of may other solutions, commercial and open source.  All of them have at least a few features that make them attractive, but when it comes right down to it, most of the commercial solutions are outrageously priced or fall short of meeting the needs of a small carrier network.

## What I want

### Template drive, object oriented configuration

Here, I'm not talking about how objects (devices) are defined, but how they are configured by the monitoring system.  Nagios wins, hands down.  A properly created template stack can make overall configuration so much easier.  Of course, moving away from hand-edited configuration files might just make this a moot point.  Nevertheless, I like being able to update a single template and instantly change how 100's or even 1000's of hosts or services are monitored.

### Recursive host discovery

I like the idea of having a process that periodically crawls the network and finds new things.  LibreNMS does this very well and I will likely examine how they do it.  However, I think I might be a little more conservative on what happens after discovery.  At the very least, the automatic generation of network maps would be most awesome!

Zabbix also has a host discovery process, but so far, I've yet to figure out how to combine discovery processes for heterogeneous networks (different types of devices from different vendors).

### Low level discovery

I do like how Zabbix uses templates to discover interfaces and services on devices (discovered or manually added).  It's a complete(?) solution with items, triggers, and graphs.

### Service Monitoring

Not much to say here.  The existing platforms pretty much have this nailed.  I really can't see doing service monitoring much differently than everyone else.  Every service is attached to a host and either it's working or it's not.  Complex service dependencies will not likely be different either.

### Interface monitoring

One thing that seems to apply across the board, is the (frontend) complexity of monitoring interfaces.  Just basic monitoring of a single interface requires a minimum of 4 OIDs be checked (ifOperStatus, ifSpeed, ifInOctets, IfOutOctets).  Doing this on a 48 port switch is a lot of work.  Doing this on an OLT with 4 GPON interfaces, each with 32 ONTS, each with 2 Ethernet interfaces means that we're processing over 1000 SNMP values.  Add in things like checking for light level, errors, duplex, and we're now talking over 5,000 values.  My employer currently has over 10 OLTs and upwards of 20 DSLAMs.  That's a LOT of interfaces to monitor.

I don't know where I'm going with this.  I know I want to obtain this data in bulk (bulkget/bulkwalk), but as for processing it, there are a plethora of methods to choose from.

### Element Management (Layer 1)

Another feature lacking in all (most?) platforms, is element management.  Not only do I want to know that two devices are connected to each other, I want to know *how* they are connected.  Copper?  Fiber? PON? Wireless?

Which cable and pair is that DSL or Fiber customer connected to?  Where are the splice points and splitters?  What wires/fibers are spliced and to where?  Layer 1 can't really be monitored, but it can be documented.

## Topology Management

One of the biggest failures of every monitoring system I've looked at, is the inability to understand complex network typologies.  Of course, Zabbix and LibreNMS don't even do parent/child dependencies out of the box, and Nagios would melt down if you tried to get it to properly monitor an ERPS ring or a multi-homed network.

Towards that end, I'm thinking to use a Directed Graph to manage the topology.  This doesn't just support Parent/Child relationships, but mandates and enforces it (or something like it).

From [Wikipedia](https://en.wikipedia.org/wiki/Directed_graph):
In [mathematics](https://en.wikipedia.org/wiki/Mathematics "Mathematics"), and more specifically in [graph theory](https://en.wikipedia.org/wiki/Graph_theory "Graph theory"), a **directed graph** (or **digraph**) is a [graph](https://en.wikipedia.org/wiki/Graph_(discrete_mathematics) "Graph (discrete mathematics)") that is made up of a set of [vertices](https://en.wikipedia.org/wiki/Vertex_(graph_theory) "Vertex (graph theory)") connected by [edges](https://en.wikipedia.org/wiki/Edge_(graph_theory) "Edge (graph theory)"), where the edges have a direction associated with them.

I don't know if the is the best approach, but it's a place to start.

### Passive Nodes

It's impossible to monitor passive nodes, such as splitters, hubs, unmanaged switches, and others.  My initial thought is to allow for both passive and active nodes.  This would also allow us to treat a 3rd party (Layer 1/2/3) provider as a passive node in our network.

### Active Nodes

At a minimum, active nodes will be required to have at least one service and one interface, even if that service is a simple ICMP echo reply to an IP address on the interface.

### Stacked Nodes

In some cases, such as with aggregation switches, it's possible to stack the devices in such a way that they appear on the network as a single device with multiple shelves, cards, modules, and interfaces.  This is where we might also consider bringing some of our passive nodes into the stack, as a GPON interface connects to a splitter, which is effectively part of stack, even if its' located several miles away.

### Nested/Virtual Nodes

I'm not sure if this is even necessary, but I would like to figure out a way to document and monitor virtual nodes, such as Virtual Machines, docker instances, jails, VRF routers, etc...

### Edges (Links)

A link is used to connect two nodes.  This might be a singular cable (that we really don't care about), or a set of wires or fibers within a cable.  To meet the goal of element management, I want to collect quite a bit of information:

 - Where's the ditch or poles?
 - How much conduit in the ditch?
 - How many cables in the conduit or on the poles?
 - How many tubes/bundles in the cable?

Of course, I will want to accommodate any type of link imaginable.  Ethernet, Fiber, DSL, Wireless, etc...

### Virtual Edges

I haven't prevsiously considered this, but it would be nice to also have a way to document and monitor virtual edges, such as MPLS networks and OSPF virtual links.

## The Backend

### Architecture

With the idea that we're looking at using a directed graph to manage our topology,  the distance between any two nodes can be easily calculated.  Towards this end, I'm thinking that taking a decentralized approach might be the best way forward.  It may even be possible to leverage block chain technology to create a decentralized global monitoring system for very large networks with strict accountability and SLA reporting capabilities.

Using Nagios as an example, we can use NSCA to submit passive check results back to the parent process or NRPE to execute plugins on remote hosts.  What if all hosts were actively involved in the process and automatically distributed the workload?

While I wouldn't impose strict limitations on the roles any one node can perform, I do want the ability to designate/prohibit certain nodes for/from certain tasks or roles.  Generally speaking though, when it comes to collecting and processing data, there's no reason not to divy up the work, especially when it comes to discovery.

In my vision of a self healing monitoring network, nodes can come and go without impacting overall performance or reliability.  SLA-related data can be replicated across the network and never lost.  With careful consideration, time series data can also be replicated to select hosts and also never lost.  Ultimately, the NMS "servers" can be considered completely disposable.

### Interaction

I'm currently leaning towards the implementation of a REST API to communicate with the node and incorporating a simple HTTP/S server to access the front end via a web browser.

I would like to see a universal front end, accessible from the desktop and mobile.

With regards to interacting with the server, I'm leaning towards a relatively simple REST API.

On the frontend, I want something that's going to work regardless of platform.  Towards that end, I'm leaning strongly towards the Quasar Framework based on Vue.js and incorporating other libraries as needed to provide a useful and beautiful UI.
