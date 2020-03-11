---
title: Link Layer
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  computer_network:
   parent: Concepts
   weight: 5
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 4
---

- understand principles behind data link layer services:
  - error detection, correction
  - sharing a broadcast channel, multiple access (无线网络)
  - link layer addressing (48 bit)
  - reliable data transfer, flow control
- instantiation and implementation of various link layer technologies

## Introduction

<img src="https://i.loli.net/2019/04/06/5ca85427a4c9a.png" width="400px"/>

- hosts and routers are nodes
- communication channels that connect adjacent nodes along communication path are links (wired links, wireless links, LANs)
- layer-2 packet is a frame, encapsulates datagram
- data-link layer has a responsibility of transferring datagram from one node to adjacent node over a link
- datagram transferred by different link protocols over different links
- each link protocol provides different services (may or may not provide rdt(reliable data service) over link)

### Services

- Framing, link access
  - encapsulate datagram into frame, adding header, trailer
  - channel access if shared mediums
  - "MAC" (media access control protocol) addresses used in frame headers to identify source, destination
- reliable delivery between adjacent nodes
  - TCP
  - seldom used on low bit-error link(fiber, some twisted pair)
  - wireless links high error rates
- flow control: pacing between adjacent sending and receiving nodes
- error detections:
  - error caused by  noise
  - receiver detects presence of errors
- error correction: receiver identifies and corrects bit error(s) without resorting to retransmission
- half-duplex and full-duplex
  with half duplex, nodes at both ends of link can transmit, but not at same time

### Where is link layer implemented

<img src="https://i.loli.net/2019/04/06/5ca858ae361ff.png" width="400px"/>

- in each and every host
- link layer implemented in "adaptor" (network interface card)
  - ethernet card, PCMCI card, 802.11 card
  - implement link, physical layer
- attaches into host's system buses
- combination of hardware, software, firmware
- sending side : encapsulates datagram in frame, address error checking bits, rdt, flow control, etc
- receiving side: look for errors, rdt, flow control etc, extracts datagram, passes to upper layer at receiving side

## Error detection and error correction

- EDC: Error detection and correction bits (redundancy)
- D: data protected by error checking, may include header files
- Error detection not 100% reliable
  - protocol may miss some errors, but rarely
  - larger EDC field yields better detection and correction

<img src="https://i.loli.net/2019/04/06/5ca85a9c09874.png" width="400px"/>

### Parity Checking

- Single Bit Parity (Detect single bit errors)
<img src="https://i.loli.net/2019/04/06/5ca85b23a266e.png" width="400px"/>
- Two dimensional Bit Parity (Detect and correct single bit errors)
<img src="https://i.loli.net/2019/04/06/5ca85b845aa63.png" width="400px"/>

### Internet Checksum

- detect "errors" (e.g. flipped bits) in transmitted packet (used at transport layer only)
- Sender:
  - treat segment contents as sequence of 16-bit integers
  - checksum: addition (1's complement sum) of segment contents
  - sender puts checksum value into UDP checksum fields
- Receiver:
  - compute checksum of received checksum
  - check if computed checksum equals checksum field value

### Cyclic Redundancy Check

- view data bits D as a binary number
- choose r+1 bit pattern (generator) G
- goal: choose r CRC bits, R, such that
  - <D,R> exactly divisible by G
  - receiver knows G, divides <D,R> by G
  - can detect all burst errors less than r+1 bits
- widely used in practice (Ethernet, 802.11 WIFI, ATM)
<img src="https://i.loli.net/2019/04/06/5ca85e5f9fc02.png" width="400px"/>
<img src="https://i.loli.net/2019/04/06/5ca85eca9089e.png" width="400px"/>

## Multiple Access Links and Protocols

- Two types of "links"
  - Point-to-point: PPP for dial-up access, point-to-point link between Ethernet switch and host
- broadcast (shared wire or medium)
  - old-fashioned Ethernet
  - upstream HFC (hybrid fiber coax)
  - 802.11 wireless LAN
  <img src="https://i.loli.net/2019/04/06/5ca85ff5c1043.png"/>
- single shared  broadcast channel
- two or more simultaneous transmissions by nodes: collision if node receives two or more signals at the same time
- multiple access protocol:
  - distributed algorithm that determines how nodes share channel, i.e. determine when node can transmit
  - communication about channel sharing must use channel itself, no out-of-band channel for coordination

### Ideal Multiple Access Protocol

Broadcast channel of rate R bps

1. when one node wants to transmit, it can send at rate R
2. when M nodes want to transmit, each can send at average rate R?M
3. fully decentralized
    - no special node to coordination transmissions
    - no synchronization of clocks, slots
4. simple

### MAC protocols: a taxonomy

#### Channel Partitioning: TDMA/ FDMA

- TDMA: time division multiple access
  - access to channel in "rounds"
  - each station gets fixed length slot(length= packet transmission time) in each round
  - unused slots go idle
- FDMA: frequency division multiple access
  - channel spectrum divided into frequency bands
  - each station assigned fixed frequency band
  - unused transmission time in frequency bands go idle
<img src="https://i.loli.net/2019/04/06/5ca86e895021e.png" width="400px"/>

#### Random access

channel not divided, allow collisions, "recover" from collisions

- when node has packet to send, transmit at full channel data rate R
- two or more transmitting nodes $\rightarrow$ collision
- random access MAC protocol specifies
  - how to detect collisions
  - how to recover from collisions (e.g. via delayed retransmission)
- examples of random access MAC protocols: slotted ALOHA, CSMA, CSMA/CD (ethernet), CSMA/CA

##### Slotted ALOHA

<img src="https://i.loli.net/2019/04/06/5ca86ff170d36.png" width="400px"/>
- all frames same size
- time divided into equal size slots (time to )

- Operation: when node obtains fresh frame, transmits in next slot
  - if no collision:  
  - if collision: node retransmits frame in each subsequent slot with pro
- Cons:
  - collisions, wasting slots
  - idle slots
  - nodes may be able to detect collision in less than time to transmit pocket
  - clock synchronization
- Pros
  - single
  - highly decentralized
  - simple
- Efficiency: long-run fraction of successful slots (many nodes, all with many frames to send)
  - suppose: N nodes with many frames to send, each transmits in slot with probability p
  - prob that given node has success in a slot = $p(1-p)^{N-1}$
  - prob than any node has a success = $Np(1-p)^{N-1}$
  - for many nodes, max efficiency $p = 1/e = 0.37$

##### Pure(unslotted) ALOHA

- unslotted Aloha: simpler, no synchronization
- when frame first arrives: transmit immediately
- collision probability increases, frame set at $t_0$ collides with other frames send in $[t_0-1, t_0+1]$
<img src="https://i.loli.net/2019/04/06/5ca87295cba56.png" width="400px"/>
- prob $p*(1-p)^{2(N-1)}$
- for many nodes, max efficiency $p = 1/(2e) = 0.18$

##### CSMA (Carrier Sense Multiple Access)

CSMA: listen before transmit, if channel sensed idle transmit entire frame, if channel sensed busy, defer transmission

<img src="https://i.loli.net/2019/04/06/5ca87401d7b82.png" width="400px"/>

- collision: entire packet transmission time wasted
- role of distance & propagation delay in determining collision probability

##### CSMA/CD (collision detection)

<img src="https://i.loli.net/2019/04/06/5ca8753567677.png" width="400px"/>

- collision detection:
  - easy in wried LANS: measure signal strengths, compare transmitted, received signals
  - difficult in wireless LANs; received signal strength overwhelmed by local transmission strength
  - CSMA/CD used in Ethernet
  - CSMA/CA (collision avoidance) used in 802.11

#### "Taking Turns" MAC protocols

- channel partitioning MAC protocols:
  - share channel efficiently and fairly at high load
  - inefficient at low load: delay in channel access, 1/N bandwidth allocated even if only 1 active node
- Random access MAC protocols:
  - efficient at low load: single node can fully utilized channel
  - high load: collision overhead
- taking turns protocols: look for best of both worlds
- Polling
  - master node "invites" slave nodes to transmit in turn
  - typically used with "dumb"
- Token passing (Taking Turns):
  - control token passed from one node to next sequentially
  - token message
  - concerns: token overhead, latency, singe point of failure
- example: Bluetooth, FDDI, IBM Token Ring

## Link-Layer Addressing

- 32-bit IP address:
  - network-layer address
  - used to get datagram to destination IP subnet
- MAC (or LAN or physical or Ethernet) address:
  - function: get frame from one interface to another physically-connected interface (same network)
  - 48 bit MAC address (for most LANS)
  - burned in NIC ROM, also sometimes software settable
- Each adapter on LAN has unique LAN address (FF-FF-FF-FF-FF-FF: broadcast address)
- mac address allocation administered by IEEE
- manufacturer buys portion of MAC address space (to assure uniqueness)
- MAC flat address $\rightarrow$ portability, can move LAN card from one LAN to another
- IP hierarchical address not portable

### ARP: address Resolution Protocol

- how to determine MAC address of B knowing B's IP address?
- Each IP node (host, router) on LAN has ARP table
- ARP table: IP/MAC address mappings for some LAN nodes, TTL(time to live): time after which address mapping will be forgotten (typically 20 minutes)
<img src="https://i.loli.net/2019/04/06/5ca87cadbddda.png" width="500px"/>
- procedure
  - A wants to send datagram to B, and B's MAC address not in A's ARP table
  - A broadcast ARP query packet, containing B's IP address, all machines on LAN receive ARP query
  - B receivers ARP packet, replies to A with its (B's) MAC address (frames send to A's MAC address(unicast))
  - A caches (saves) IP-to-MAC address pair in its ARP table until information becomes old (times out)
  - ARP is "plug-and-play", node creates their ARP tables without intervention form net administrator
- example:
<img src="https://i.loli.net/2019/04/06/5ca87f2ac983f.png" width="550px"/>

## Ethernet

- features:
  - cheap
  - first widely used LAN technology
  - simpler, cheaper than  token LANs and ATM
  - kept up with speed race: 10Mbps - 10 Gbps
- bus topology popular through mid 90-s (all nodes in same collision domain, can collide with each other)
- today: star topology prevails
  - active, switch in center
  - each "spoke" run a (separate) Ethernet protocol (nodes do not collide with each other)
- Ethernet Frame structure
<img src="https://i.loli.net/2019/04/06/5ca88188bf408.png" width="500px"/>
  - Preamble: 7 bytes with pattern 10101010 followed by one byte with pattern 10101011
  - used to synchronize receiver, sender clock rates
  - Addresses: 6 bytes, if adapter receives frame with matching destination address, or with broadcast address (e.g. ARP packet), it passes data in frame to network layer protocol, otherwise, adapter discards frame
  - Type: indicates higher layer protocol (mostly IP but others possible, e.g. Novell IPX, Apple Talk)
  - CRC: checked at receiver, if error is detected, frame is dropped
- Unreliable, connectionless
  - connectionless: No handshaking between sending and receiving NICs
  - unreliable： receiving NIC(network interface card) doesn't send ACKs or NAKs to sending NIC
    - stream of datagrams passed to network layer can have gaps(missing datagrams)
    - gaps will be filled if app is using TCP, otherwise app will see gaps
  - Ethernet's MAC protocol: unslotted CSMA/CD

### Ethernet CSMA/CD algorithm

1. NIC receives datagram from network layer, creates frame
2. IF NIC senses channel idle, starts frame transmission, IF NIC senses channel busy, waits until channel idle, then transmits
3. If NIC transmits entire frame without detecting another transmission, NIC is done with frame
4. If NIC detects another transmission while transmitting, aborts and sends jam signal
5. After aborting, NIC enters exponential backoff: after mth collision, NIC chooses K at random from $(0, 1,2, ..., 2^m-1)$ NIC waits K*512 bit times, returns to Step2

- Jam signal: make sure all other transmitters are aware of collision: 48 bits
- Bit time: 1 microsecond for 10 Mbps Ethernet, for K=1023,wait time is about 50 msec
- Exponential Backoff (最多连续16次)
  - Goal: adapt retransmission attempts to estimated current load (heavy load: random wait will be longer)
  - first collision: choose K from (0,1), delay is K*512 bit transmission times
  - after second collision: choose K from {0,1,2,3}
  - after ten collisions: choose K from {0,1,2,3,4,...,1023}
- Efficiency (70%)
  - $T_{prop}$ = max prop delay between 2 nodes in LAN
  - $T_{trans}$ = time to transmit max-size frame
  $$\text{efficiency} = \frac{1}{1+5t_{prop}/t_{trans}}$$
  - Efficiency goes to 1 as $t_{prop}$ goes to 0, as t_{trans}$ goes to infinity
  - better performance than ALOHA: and simple, cheap, decentralized

### 802.3 Ethernet Standards: Link & Physical Layers

- many different Ethernet standards
  - common MAC protocol and frame format
  - different speeds: 2Mbps, 10Mbps, 100Mbps, 1Gbps, 10Gbps
  - different physical layer media: fiber, cable
  <img src="https://i.loli.net/2019/04/06/5ca889e6adb92.png"/>
- Manchester encoding
  - used in 10Base T
  - each bit has a transition
  - allows clocks in sending and receiving nodes to synchronize to each other, no need for a centralized, global clock among nodes

## Link-Layer switches

- Hubs
  - physical-layer ("dumb" repeaters, 只处理信号)
  - bits coming in one link go out all other links at same rate
  - all nodes connected to hub can collide with one another
  - no frame buffering
  - no CSMA/CD at hub: host NIC detect collisions
- Switch: allow multiple simultaneous transmissions
  - link-layer device: smarter than hubs, take active role
    - store, forward Ethernet frames
    - examine incoming frame's MAC address, selectively forward frame to one-or-more outgoing links when frame is to be forwarded on segment, uses CSMA/CD to access segment
  - transparent, hosts are unaware of presence of switches
  - plug-and-play, self-learning, switch do not need to be configured
  - switch table:  Each switch has a switch table, each entry (MAC address of host, interface to reach host, time stamp

### Self-Learning

1. Switch table is initially empty
2. For each incoming frame received on an interface, the switches stores in its table
    1. The MAC address in the frame's source address field
    2. the interface from which the frame arrived
    3. the current time
3. the switch deletes an address in the table if no frames are received with that address as the source address after some period of time

### Filtering/forwarding

- Filtering: whether a frame should be forwarded to some interface or should just be dropped
- Forwarding: determine the interfaces to which a frame should be directed, and then moves the frame to those interfaces
- suppose a frame with destination address DD-DD-DD-DD-DD-DD arrives at the switch on interface x
  - there is no entry in the table for DD-DD-DD-DD-DD-DD, the switch forwards copies of the frame to the output buffers preceding all interfaces (broadcast the frame)
  - there is an entry in the table, associating DD-DD-DD-DD-DD-DD with interface x: there is no need to forward the frame to any of the other interfaces, switch performs filtering function by discarding the frame. (送和收在同一区域）
  - there is an entry in table, associating DD-DD-DD-DD-DD-DD with interface $y \neq x$, the frame needs to be forwarded to the LAN segment attached to interface y. Switch performs its forwarding function by putting the frame in an output buffer that precedes interface y.

### Switches vs. Routers

<img src="https://i.loli.net/2019/04/06/5ca8921f69722.png" width="400px"/>

- both store-and-forward devices: routers are network layer devices, switches are link layer address
- Routers maintain routing tables, implement routing algorithms
- switches maintain switch tables, implement filtering, learning algorithms

## PPP [RFC 1557]

Point to Point Data Link Control

- one sender, one receiver, one link: easier than broadcast link
- no media Access control
- no need for explicit MAC addressing  (e.g. dialup link)
- popular point-to-point DLC protocols: PPP, HDLC

### PPP Frame

- packet framing: encapsulation of network-layer datagram in data link frame
  - carry network layer data of any network layer protocol (not just IP) at same time
  - ability to demultiplex upwards
- bit transparency: must carry any bit pattern in the data field
- error detection (no correction)
- connection liveness: detect, signal link failure to network layer
- network layer address negotiation: endpoint can learn/configure each other's network address
- data frame
    <img src="https://i.loli.net/2019/04/07/5ca9ba6436d50.png" width="450px"/>
  - Flag: delimiter (framing)
  - Address: does nothing
  - control: does nothing:
  - protocol: upper layer protocol to which frame delivered (e.g. PPP-LCP, IP, IPCP etc)
  - info: information
  - check: cyclic redundancy check for error-detection
- PPP non-requirements
  - no error correction
  - no flow control
  - out of order delivery
  - no need to support multipoint links
  - error recovery, flow control, data re-ordering all relegated to higher layers
- Byte Stuffing
  - "data transparency" requirement: data field must be allowed to include flag pattern  <01111110>
  - Sender: adds extra <01111110> byte after each <01111110> data byte
  - Receiver: two 01111110 bytes in a row: discard first byte, continue data reception; singe 01111110:flag byte
- PPP data Control Protocol
  - before exchanging network-layer data, data link peers must
  - configure PPP link (max frame length, authentication)
  - learn / configure network layer information

### ATM

- Virtualization of networks
  - Layering of abstractions: don't sweat the details of the lower layer, only deal with lower layers abstractly
  - two layers of addressing: internetwork and local network
  - new layer (IP) makes everything homogeneous at internetwork layer
- Asynchronous Transfer Mode: ATM
  - Goal: integrated, end-end transport of carry voice, video, data
  - meeting timing / QoS requirements of voice, video (versus Internet best-effort model)
  - "next generation" telephony (下一代网络电话）
  - packet-switching (fixed length packets, 53 bytes, called "cells") using virtual circuits
- For more information, see [Computer Networking Website](https://www.net.t-labs.tu-berlin.de/teaching/computer_networking/05.09.htm)

### Multi-protocol label switching (MPLS)

- Goal: speed up IP forwarding by using fixed length label (instead of IP address) to do forwarding (20bits)
  - Borrowing ideas from Virtual Circuit (VC) approach
  - but IP datagram still keeps IP address
   <img src="https://i.loli.net/2019/04/07/5ca9bfc799f05.png" width="500px"/>
- MPLS-enhanced forwarding
   <img src="https://i.loli.net/2019/04/07/5ca9c076ea536.png" width="500px"/>