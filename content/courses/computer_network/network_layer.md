---
title: Network Layer
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  computer_network:
   parent: Concepts
   weight: 4
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 3
---

## Introduction

- transport segment from sending to receiving host
- on sending side encapsulates segments into datagrams
- on receiving side, delivers segments to transport layer
- network layer protocols in every host, router
- router examines header fields in all IP datagrams passing through it

### Two keys

- Forwarding: move packets from router's input to appropriate router output
<img src="https://i.loli.net/2019/04/03/5ca48fce906b6.png" width="450px"/>
- Routing: determine route taken by packets from source to destination, routing algorithm

### Connection setup

- 3rd important function in some network architectures: ATM, frame relay, X.25
- before datagrams flow, two end hosts and intervening routers establish virtual connection (routers get involved)
- network vs transport layer connection service:
  - network: between two hosts(may also involve intervening routers in case of VCs)
  - transport: between two processes

### Network Service model

- example services for individual datagrams: guaranteed delivery, guaranteed delivery with less than 40 milliseconds delay
- example services for a flow of datagrams: in order datagram delivery, guaranteed minimum bandwidth to flow,  restrictions on change in inter-packet spacing (封包和封包的间隔的变化(时间差), 变异度)
- 建连线有上面的这些机制

<img src="https://i.loli.net/2019/04/03/5ca492c7dbe88.png" width="650px"/>

## Virtual circuit and datagram networks

### connection and connection-less service

- datagram network provides network-layer connectionless service
- VC network provides network-layer connection service
- analogous to transport-layer services, service: host-to-host; no choice: network providers one or the other(只能选一种); implementation: in network core

### Virtual circuits

- source-to-dest path behaves like telephone circuit (performance-wise, network actions along source-to-dest path)
- call setup, teardown for each call before data can flow
- each packet carries VC identifier (not destination host address)
- every router on source-dest path maintains "state" for each passing connection
- link, router resources(bandwidth, buffers) may be allocated to VC (dedicated resources = predictable service)
<img src="https://i.loli.net/2019/04/03/5ca4968c668c7.png" width="400px"/>
<img src="https://i.loli.net/2019/04/03/5ca4969aecdd0.png" width="400px"/>

### VC implementation

- VC consists of
  - path from source to destination
  - VC numbers, one number for each link along path
  - entries in forwarding tables in routers along path
- packet belonging to VC carries VC number(rather than destination address)
- VC number can be changed on each link (New VC number comes from forwarding table)
<img src="https://i.loli.net/2019/04/03/5ca4981ed5b64.png" width="400px"/>

### Datagram Networks

- no call setup at network layer
- routers: no state about end-to-end connections, no network-level concept of "connection"
- packets forwarded using destination host address, packets between some source-destination pair may take different paths
<img src="https://i.loli.net/2019/04/03/5ca4995db5593.png" width="400px"/>
- forwarding table
  - longest prefix matching
  <img src="https://i.loli.net/2019/04/03/5ca49a3b909dd.png" width="600px"/>

### Datagram or VC network: Why

- Internet (datagram)
  - data exchange among computers, elastic service, no strict timing request
  - smart end systems (computer), can adapt, perform control error recovery, simple inside network, complexity at "edge"
  - many link types, different characteristics, uniform service difficult
- ATM (VC)
  - evolved from telephony
  - human conversation, strict timing, reliability requirements, need for guaranteed service
  - "dumb" end systems (telephones, complexity inside network)

## What's inside a router

<img src="https://i.loli.net/2019/04/03/5ca4a00b51702.png" width="400px"/>
- routing algorithms / protocol (RIP, OSPF, BGP)
- forwarding algorithms from incoming to outgoing link

### Input Port Functions

<img src="https://i.loli.net/2019/04/03/5ca4a134239c6.png" width="400px"/>

- given datagram destination, lookup output port using forwarding table in input port memory
- goal: complete input port processing at line speed
- queuing: if datagrams arrive faster than forwarding rate into switch fabric

### Switching fabrics

<img src="https://i.loli.net/2019/04/03/5ca4a2e556669.png" width="400px"/>

- switching via a bus:
  - datagram from input port memory to output port memory via a shared bus
  - bus connection: switching speed limited by bus bandwidth
  - 32 Gbps bus, Cisco 5600: sufficient speed for access and enterprise routers
- switching through a crossbar
- switching via An interconnection network
  - overcome bus bandwidth limitations
  - Banyan networks, other interconnection nets initially developed to connect processors in multiprocessor
  - advanced design: fragmenting datagram into fixed length cell

### Output ports

<img src="https://i.loli.net/2019/04/03/5ca4a43acefdc.png" width="400px"/>

- buffering: required when datagrams arrive from fabric faster than the transmission rate
- scheduling discipline: chooses among queued datagrams for transmission
<img src="https://i.loli.net/2019/04/03/5ca4a4d9c5b79.png" width="400px"/>
- queueing (delay) and loss due to output port buffer overflow
- How much buffering
  - RFC 3439 rule of thumb: average buffering equal to "typical" RTT (250 millisecond) times link capacity C (C = 10 Gpbs link, 2.5 Gbit buffer)
  - Recent recommendation: with N flows, buffering equal to $RTT*C/(\sqrt(N))$
- Input port Queuing problem
<img src="https://i.loli.net/2019/04/03/5ca4a677b909c.png" width="400px"/>
  - HOL (head-of-the-line) blocking

## IP: Internet Protocol

<img src="https://i.loli.net/2019/04/03/5ca4a7516558a.png" width="400px"/>

### IP datagram format

<img src="https://i.loli.net/2019/04/03/5ca4a861461f7.png" width="400px"/>

- 16-bit identifier, flags, 13-bit fragmentation offset: fragmentation/reassembly
- time to live: max number remaining hops (decremented at each router)
- upper layer: upper layer protocol to deliver
- header checksum

#### IP Datagram Fragmentation

<img src="https://i.loli.net/2019/04/03/5ca4a906a20cf.png" width="400px"/>

<img src="https://i.loli.net/2019/04/03/5ca4a9b07232e.png" width="600px"/>

### IPv4 addressing

- IP address: 32-bit identifier for host, router interface
- interface: connection between host/router and physical link
  - router's typically have multiple interfaces
  - host typically has one interface
  - ip address associated with each interface

#### Subnets

- subnet part (high order bits)
- host part (low order bits)
- device interfaces with same subnet part of IP address
- can physically reach other without intervening router
<img src="https://i.loli.net/2019/04/03/5ca4ace603a45.png" width="400px"/>
- to determine the subnets, detach each interface from its host or router, creating islands of isolated networks, each isolated network is called a subnet
- six subnets
  <img src="https://i.loli.net/2019/04/03/5ca4ade4aa342.png" width="400px"/>
- CIDR: Classless Inter Domain Routing
  - subnet portion of address of arbitrary length
  - address format: a.b.c.d/x where x is number bits in subnet portion of address
- Get IP
  - hard-coded by system admin in a file,  `unix: /etc/rc.config`
  - DHCP: Dynamic Host Configuration Protocol: dynamically get address from as server, "plug-and-play"
- Get subnet part of IP address: Gets allocated portion of its provider ISP's address space
- ISP get block of address? ICANN (Internet Corporation for Assigned Names and Numbers)

#### DHCP: Dynamic Host Configuration Protocol

- Goal: allow host to dynamically obtain its IP address from network server when it joins network, can renew its lease on address in use, allows reuse of address (only hold address while connected an "on"), support for mobile users who want to join network (more shortly)
- DHCP overview:
  - host broadcasts "DHCP discover" message
  - DHCP server respond with "DHCP offer" message
  - host requests IP address "DHCP request" message
  - DHCP server sends address "DHCP ACK" message
<img src="https://i.loli.net/2019/04/03/5ca4b5f6e31b0.png" width="400px"/>
<img src="https://i.loli.net/2019/04/03/5ca4b605e3bf1.png" width="450px"/>

#### Hierarchical addressing

<img src="https://i.loli.net/2019/04/04/5ca5a81531772.png" width="400px"/>
<img src="https://i.loli.net/2019/04/04/5ca5a7df7e3e2.png" width="400px"/>
more specific route
<img src="https://i.loli.net/2019/04/04/5ca5a8c57f578.png" width="400px"/>

#### Network Address Translation (NAT)

- Motivation: local network use just one IP address as far as outside world is concerned:
  - range of addresses not needed from ISP: just one IP address for all devices
  - can change addresses of devices in local network without notifying outside world
  - can change ISP without changing addresses of devices in local network
  - devices inside local net not explicitly addressable, visible by outside world (a security plus)
- Implementation: NAT router must:
  - outgoing datagrams: replace (source IP address, port number) of every outgoing datagram to (NAT IP address, new port number), remote clients/ servers will respond using (NAT IP address, )

<img src="https://i.loli.net/2019/04/04/5ca5ac6370caf.png" width="500px"/>

- 16 bit port-number field (65536)
- NAT is controversial
  - routers should only process up to layer 3 (but involved layer 4)
  - violates end-to-end argument, NAT possibility must be taken into account by app designers (eg. P2P applications)
  - address shortage should instead be solved by IPv6
- NAT traversal problem
  - client wants to connect to server with address 10.0.0.1, server address 10.0.0.1 local to LAN (client can't use it as destination address), only one externally visible NAT address
  - solution 1: statically configure NAT to forward incoming connection requests at given port to server (e.g. 123.76.29.7 port 2500 always forwarded to 10.0.0.1 port 2500)
  - solution2: Universal Plug and Play (UPnP),  Internet Gateway Device(IGD) protocol. Allows NATed host to:
  - learn public IP address
  - add /remove port mappings (with lease times)
  - automate static NAT port map configuration
  - solution3: relaying (used in Skype)
    - NATed client establishes connection to relay
    - external client connects to relay
    - relay bridges packets between connections

### ICMP (Internet Control Message Protocol)

- used by host & routers to communicate network-level information
  - error reporting
  - unreachable host, network, port, protocol
  - echo request/reply (used by ping)
- network-layer above "IP", ICMP messages carried in IP datagrams
- ICMP message: type, code plus first 8 bytes of IP datagram causing error

#### Traceroute and ICMP

- source sends series of UDP segment to destination
  - first has TTL=1
  - second has TTL=2, etc
  - unlikely port number
- when nth datagram arrives to nth router
  - router discards datagram
  - and sends to source an ICMP message (type 11, code 0)
  - Message includes name of routers & IP address
- when ICMP message arrives, source calculates RTT
- traceroute does this 3 times
- Stoping criterion
  - UDP segment eventually arrives at destination host
  - destination returns ICMP "port unreachable" packet (type 3, code 3)
  - when source gets this ICMP, stops

### IPv6

- Initial motivation: 32-bit address space soon to be completely allocated
- additional motivation:
  - header format helps speed processing/forwarding
    - header changes to facilitate QoS
    - IPv6 datagram format: fixed-length 40 byte header, no fragmentation allowed
<img src="https://i.loli.net/2019/04/04/5ca5e8da43dfe.png" width="400px"/>
- priority: identify priority among datagrams in flow
- flow label: identify datagrams in same "flow"
- next header: identify upper layer protocol for data
- checksum: removed entirely to reduce processing time at each step
- options: allowed, but outside of header, indicated by "Next Header" field
- ICMPv6: new version of ICMP, additional message types ("packet too big"), multicast group management functions

#### Transition From IPv4 to IPv6

- not all routers can be upgraded simultaneous, no "flag days"
- operate mixed IPv4 and IPv6 routers: **Tunneling**, IPv6 carried as payload in IPv4 datagram among IPv4 routers
<img src="https://i.loli.net/2019/04/04/5ca5eacac3bed.png" width="400px"/>

## Routing Algorithm

- Graph abstraction
    <img src="https://i.loli.net/2019/04/04/5ca5ebfacbbcd.png" width="400px"/>
- Routing Algorithm: algorithm that finds least-cost path
- Global: all routers have complete topology, link cost info (link-state algorithm)
- Decentralized
  - router knows physically-connected neighbors, link costs to neighbors
    - iterative process of computation, exchange info with neighbors
    - "distance vector" algorithms
- static or dynamic?
  - static: routes change slowly over time
  - dynamic: routes change more quickly (periodic update, in response to link cost changes)

### Link-State Routing Algorithm

Dijkstra's algorithm (See Algorithm)

- Oscillations with congestion-sensitive routing (link cost = amount of carried traffic)
<img src="https://i.loli.net/2019/04/06/5ca8185104528.png" width="500px"/>

### The Distance-Vector (DV) Routing Algorithm

Distance Vector Algorithm (see Algorithm)

- Node x maintains distance vector
- Node x also maintains its neighbors' distance vectors
- Bellman-Ford equation (dynamic programming)
$d_x(y) = min_v (c(x,v) + d_v(y))$ where min is taken over all neighbors v of x
<img src="https://i.loli.net/2019/04/04/5ca5f6a51904f.png" width="500px"/>
- good news travels fast, bad news travels slow
<img src="https://i.loli.net/2019/04/06/5ca8155a2ec9c.png" width="400px"/>
  - Figure b needs 44 iterations before algorithm stabilizes
- Poisoned reverse
  - If Z routes through Y to get to X: Z tells Y its (Z's) distance to X is infinite (So y won't route to X via Z)
  - loops involving three or more nodes (rather than simply two immediately neighboring nodes) will not be detected by the poisoned reverse technique

### Comparison of LS and DV

- Message complexity
  - LS: with n nodes, E links O(nE) message send
  - DV: exchange between neighbors only, convergence time varies
- Speed of Convergence
  - LS: $O(n^2)$ algorithm requires $O(nE)$ messages, may have oscillations
  - DV: convergence time varies, may be routing loops, count-to-infinity
- Robustness: what happens if router malfunctions?
  - LS: node can advertise incorrect link cost, each node computes only its own table
  - DV node can advertise incorrect path cost, each node's table used by others, error propagate through network

### Hierarchical routing

- scale: with 200 million destinations:
  - can't store all destination's in routing tables
  - routing table exchange would swamp links
- administrative autonomy
  - internet = network of networks
  - each network admin may want to control routing in its own network
- aggregate routers into regions "autonomous systems (AS)"
- routers in same AS run same routing protocol
  - "intra-AS" routing protocol
  - routers in different AS can run different intra-AS running protocol
- Gateway router : direct link to router in another AS
<img src="https://i.loli.net/2019/04/04/5ca5fc1c11b2e.png" width="500px"/>
- Inter-AS tasks, suppose router in AS1 receives datagram destined outside of AS1: router should forward packet to gateway router, but which one?
  - AS1 must learn which destination are reachable through AS2, which through AS3
    - propagate this reachability info to all routers in AS1

#### Hot-potato routing

Suppose AS1 learns from inter-AS protocol that subnet X is reachable from AS3 and from AS2, to configure forwarding table, router 1d must determine towards which gateway it should forward packets for destination x.(hot-potato routing- send packet towards closest of two routers)
<img src="https://i.loli.net/2019/04/04/5ca5ff85a0ec8.png" width="600px"/>

## Routing in the Internet

Interior Gateway Protocols (IGP)

### RIP (Routing Information Protocol)

- distance vector algorithm
- included in BSD-UNIX distribution in 1982
- distance metric: number of hops (max = 15hops)
<img src="https://i.loli.net/2019/04/04/5ca600bf47feb.png" width="400px"/>
- exchanged among neighbors every 30 seconds via Response Message (also called advertisement)
- each advertisement: list of up to 25 destination subnets within AS
- Link Failure and Recovery: if no advertisement heard after 180 seconds $\rightarrow$ neighbor/link declared dead
  - routes via neighbor invalidated
  - new advertisements send to neighbors
  - neighbors in turn send out new advertisements
  - link failure info quickly (?) propagates to entire network
  - poison reverse: used to prevent ping-pong loops (infinite distance = 16 loops)
- RIP table processing
  - RIP routing tables managed by application-level process called route-d (daemon) (Send distance vector)
  - advertisements send in UDP packets, periodically repeated
<img src="https://i.loli.net/2019/04/06/5ca81b33efa24.png" width="400px"/>

### OSPF (Open Shortest Path First)

- open: publicly available
- uses Link State algorithm:
  - LS packet dissemination
  - topology map at each node
  - route computation using Dijkstra's algorithm
- OSPF advertisement carries one entry per neighbor router
- advertisements disseminated to entire AS (via flooding), carried in OSPF messages directly over IP (rather than TCP or UDP)
- advanced features
  - security: all OSPF messages authenticated(to prevent malicious intrusion)
  - multiple same-cost paths allowed (only one path in RIP)
  - For each link, multiple cost metrics for different TOS (type of service) (e.g. satellite link cost set "low: for best effort: high for real time)
  - integrated uni- and multicast support: Multicast OSPF (MOSPF) uses same topology data base as OSPF
  - hierarchical OSPF in large domains

#### BGP (Border Gateway Protocol) Inter-AS Routing

- the de facto standard
- BGP provides each AS a means to:
  - Obtain subnet reachability information from neighboring ASs
  - Propagate reachability information to all AS-internal routers
  - Determine "good" routers to subnets based on reachability information and policy
- allows subnet to advertise its existence to rest of Internet
- Pairs of routers (BGP peers) exchange routing info over semi-permanent TCP connections: BGP sessions (BGP sessions need not correspond to physical links)
<img src="https://i.loli.net/2019/04/06/5ca81da7a1e5d.png" width="400px"/>
- When As2 advertise to a prefix to AS1:
  - AS2 promises it will forward datagrams towards that prefix
  - AS2 can aggregate prefixes in its advertisement

Distributing reachability information:

- Using eBGP(external) session between 3a and 1c, AS3 sends prefix reachability info to AS1
  - 1c can then use iBGP do distribute new prefix info to all routers in AS1
  - 1b can then re-advertise new reachability info to AS2 over 1b-to-2a eBGP session
- when router learns of new prefix, it creates entry for prefix in its forwarding table
- advertised prefix includes BGP attributes: prefix + "attributes" = "route"
- two important attributes
  1. AS-PATH: contains ASs through which prefix advertisement has passed
  2. NEXT-HOP: indicates specific internal-AS router to next-hop AS(may be multiple links from current AS to next-hop-AS)
- when gateway router receives route advertisement, use **import policy** to accept/decline

##### BGP route selection

- router may learn about more than 1 route to some prefix, router must select route
- elimination rules:
  1. local preference value attribute: policy decision
  2. shortest AS-PATH
  3. closest NEXT-HOP router: hot potato routing
  4. additional criteria

##### BGP messages exchanged using TCP

- OPEN:  open TCP connections to peer and authenticates sender
- UPDATE: advertises new path
- KEEPALIVE: keeps connection alive in absence of UPDATES
- NOTIFICATION: reports error in previous message also used to close connection

##### BGP routing policy

<img src="https://i.loli.net/2019/04/06/5ca820d85e63b.png" width="400px"/>

- A, B, C are provider networks
- X, W, Y are customer (of provider networks)
- X is dual-homed: attached  to two networks
  - X dose not want to route from B via X to C, so X will not advertise to B a route to C
- A advertises path AW to B
- B advertises path BAW to X
- should B advertise path BAW to C?
  - No, B gets no "revenue" for routing CBAW since neither W nor C are B's customers
  - B wants to force C route to w via A
  - B wants to route only to/from its customers

##### Different Intra- and Inter-AS routing

- Policy
  - Inter-AS: admin wants control over how its traffic routed, who routes through its net
  - Intra-AS: single admin, no policy
- Scale: hierarchical routing saves table size, reduced update traffic
- Performance:
  - Intra-AS: can focus on performance
  - Inter-AS: policy may dominate over performance

## Broadcast and multicast routing

### Broadcast Routing

- deliver packets from source to all other nodes
- source duplication is inefficient
<img src="https://i.loli.net/2019/04/06/5ca822f9dc436.png" width="400px"/>
- In-network duplication:
  - flooding: when node receives broadcast packet, sends copy to all neighbors (problem: cycles & broadcast storm)
  - controlled flooding: node only broadcast packet if it hasn't broadcast same packet before, Node keeps track of packet ids already broadcasted; Reverse Path Forwarding(RPF): only forward packet if it arrived on shortest path between node and source
  - spanning tree: no redundant packet received by any node

#### Spanning Tree

<img src="https://i.loli.net/2019/04/06/5ca82424dea45.png" width="400px"/>
- First construct a spanning tree
- Nodes forward copies only along spanning tree
- Creation:
  - center node
  - each node send unicast join message to center node
  - message forwarded until it arrives at a node already belonging to spanning tree
  <img src="https://i.loli.net/2019/04/06/5ca824d3bd778.png" width="400px"/>

### Multicast Routing: Problem statement

- find a tree (or trees) connecting routers having local multicast group members
  - tree: not all paths between routers used
  - source-based: different tree from each sender to receivers
  - shared-tree: same tree used by all group members
- sourced-based tree: one tree per source, shorted path trees, reverse path forwarding
- group-shared tree: group uses one tree (minimal spanning tree, center-based trees)

#### Shortest Path tree

  Dijkstra's algorithm

#### Reverse path forwarding

<img src="https://i.loli.net/2019/04/06/5ca827388a520.png" width="400px"/>

Pruning: forwarding tree contains subtrees with no multicast group members: no need to forward datagrams down subtree, "prune" message send upstream by router with no downstream group members.

#### Shared-Tree: Steiner Tree

- minimum cost tree connecting all routers with attached group members
- problem is NP-complete
- excellent heuristics exists (近似解)
- not used in practice
  1. computational complexity
  2. information about entire network needed
  3. monolithic(庞大): rerun whenever a router needs to join/leave

#### Center-based trees

- single delivery tree shared by all
- one router identified as "center" of tree
- to join:
  - edge router sends unicast join-message addressed to center router
  - join-message "processed" by intermediate routers and forwarded towards center
  - join-message either hits existing tree branch for this center, or arrives at center
  - path taken by join-message becomes new branch of tree for this router

#### Internet Multicasting Routing: DVMRP

- DVMRP: distance vector multicast routing protocol, RFC1075
- flood and prune: reverse path forwarding, source-based tree
- soft state: DVMRP router periodically (1min) "forgets" branches are pruned
  - multicast data again flows down unpruned branch
  - downstream router: re-prune or else continue to receive data
- routers can quickly regraft to tree, following IGMP (internet Group Management protocol) join at leaf

##### Protocol Independent Multicast: RIP

- not dependent on any specific underlying unicast routing algorithm (works with all)
- Dense: group members densely packed, in "close" proximity, bandwidth more plentiful
  - group membership by routers assumed until routers explicitly prune
  - data-driven construction on multicast tree (e.g. RPF1)
  - bandwidth and non-group-router processing profligate (浪费)
- Sparse: number of networks with group members small w.r.t. number of interconnected networks, group members "widely dispersed", bandwidth not plentiful
  - no membership until routers explicitly join
  - receiver-driven construction of multicast tree (e.g. center-based)
  - bandwidth and non-group-router processing conservative