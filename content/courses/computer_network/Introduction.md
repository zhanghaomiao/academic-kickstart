---
title: Introduction
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  computer_network:
   parent: Concepts
   weight: 1
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 
---

## Computing devices

- Hosts (= end systems)  (PC workstations, servers, Phones) running network apps
- Communication Links (fiber, copper, radio, satellite) physical link
  - Transmission rate = bandwidth
- routers: forward packets (chunks of data)
- protocols: control sending, receiving of messages (TCP, IP, HTTP, FTP, PPP)
- Internet: network of networks, loosely hierarchical, public Internet(有限的) versus private Intranet
- Internet standards (定标准的组织)
  - RFC request for comments
  - IETF: Internet Engineering Task Force
- communication infrastructure enables distributed applications (Web, email, games)
- communication services provided to applications(connectionless, connection-oriented)
- Protocol
  - all communication activity in Internet governed by protocols
  - format and order of messages sent and received among network entities as well as actions taken on the transmission/receipt of a message
- network edge : applications and hosts
- network core: routers, network of networks
- access networks(连接 edge 和 core, 连接 physical media, communication links)

## The network edge

- end systems(hosts) run application programs
- client/server model: client host requests, receives service from always-on server
- peer-peer model : minimal(no) use of dedicated server
- TCP services (Transmission Control protocol)
  - reliable, in-order byte-stream data transfer(loss: 掉的做法是 acknowledgements and retransmissions)
  - Flow control, sender won't overwhelm receiver 流量控制 (sender 可以送多快是由 receiver 决定的)
  - congestion control, senders slow down sending rate when networks congested (connection太多时， 网络发生拥挤， 会放慢速度, 逐渐增大封包量， 如果封包掉了，认为网络拥挤)
  - E.g. HTTP(web), FTP(File transfer), SMTP(email)
- UDP (User Datagram Protocol) 不需要建连线
  - unreliable data transfer
  - no flow control
  - no congestion control
  - E.g. streaming media(掉了不会影响), teleconferencing, DNS, Internet telephony

## Network access and physical media

- How to connect end systems to edge router?
  - residential access nets
  - institutional access networks (school, company)
  - mobile access networks
- keep in mind:
  - bandwidth (bits per second) of access network
  - shared or dedicated

### Residential access

- dialup via modem
  - up to 56Kbps direct access to router (often less)
  - can't surf and phone at same time
- ADSL (asymmetric digital subscriber line)
  - up to 1 Mpbs upstream
  - up to 8 Mpbs downstream
  - FDM 50KHz - 1MHz for downstream, 4KHz-50kHZ for upstream, 0kHZ-4kHZ for ordinary telephone
- HFC: hybrid fiber coaxial cable, asymmetric, up to 30 Mbps downstream, 2Mpbs upstream
  - network of cable and fiber attaches homes to ISP router, shared access to router among home, issues(congestion, dimensioning)
  - deployment: available via cable components, e.g. MediaOne

### Company access

- company/university local area network (LAN) connects end system to edge router
- Ethernet: shared or dedicated link connects end system and router(10Mbps, 100Mpbs, Gigabit Ethernet)
- deployment: institutions, home

### Wireless access networks

- shared wireless access network connects end system to router, via base station
- wireless LANs: 802.11b (WiFI): 11 Mbps, 802.11g: 54Mbps
- wider-area wireless access, provided by telecom operator, 3G/4G, WAP/GPRS

### Physical Media

- Bit: propagates between transmitter/receiver pairs
- Physical link: what lies between transmitter & receiver
  - Guided media, by solid media
    - Twisted Pairs(tow insulated copper wires)
    - Coaxial cable, two concentric copper conductors, bidirectional(baseband:single channel on cable, broadband: multiple channels on cable)
    - Fiber optic cable: glass fiber carrying light pulses, each pulse a bit, high-speed operation, low error rate: repeaters spaced far apart, immune to electromagnetic noise
  - unguided media, no physical line (effected by reflection, obstruction of objects, interference)
    - terrestrial microwave (45 Mpbs channels)
    - LAN (e.g. Wifi)
    - wide-area (e.g. cellular)
    - satellite(50 Mpbs channel, 270 millisecond end-end delay)

## Network core

- 连接成网状 mesh of interconnected routers
- how is data transferred through net?
  - Circuit Switching: dedicated circular per call: telephone net
  - packet-switching: data sent through net in discrete "chunks"

### Circuit Switching

- end to end resources reserved(保留一定的频宽)
- 保留 link bandwidth, switch capacity, divided into pieces
- dedicated resources: no sharing (别的core无法使用)
- circuit-like (guaranteed) performance
- call setup required(建连线)
<img src="https://i.loli.net/2019/03/22/5c94cfc339ab9.png" width="400px"/>
- two splits (TDM)
  <img src="https://i.loli.net/2019/03/22/5c94d0e9d088e.png" width="400px"/>

### Packet Switching

  each end-end data stream divided into packets, user A,B packets share network resources, each packet uses **full link bandwidth**, resources used as needed

- resource contention (竞争)
  - aggregate resource demand can exceed amount available
  - congestion: pockets queue, wait for link use
  - store and forward, packets move one hop at a time(先排队， 后发送), transmit over link, wait turn at next link
  <img src="https://i.loli.net/2019/03/22/5c94d24556aa0.png" width="400px"/>
  sequence of A & B packets done not have fixed pattern $\rightarrow$ statistical multiplexing, in TDM each host get some slot in revolving TDM frame
- move packets through routers form source to destination (path selection algorithms)
- datagram network:
  - destination address in packet determines next hop
  - routes may change during session
  - analogy, driving, asking directions
- virtual circuit network
  - each packet carries tag (virtual circuit ID), tag determines next hop
  - fixed path determined at call setup time, remains fixed through call
  - routers maintain per-call state (每一个标签需要记住)

### Packet Switching VS. Circuit Switching

Packet switching allows more user to use network Example: 1Mbit link,  each user 100kbps when "active", active 10% of per time

- Circuit-Switching : 10 users
- Packet-Switching: with 35 users, probability > 10 active less than 0.004
- Packet Switching: greater for bursty data, resource sharing, simpler, no call setup
- Excessive congestion: packet delay and loss, protocols needed for reliable data transfer, congestion control
- How to provide circuit-like behavior? bandwidth guarantees needed for audio/video apps, still an unsolved problem

## Internet Structure: network of network

- roughly hierarchical
- Tier 1 ISP(internet service provider) (e.g. UUNet, Sprint, AT&T) national/international coverage (treat each other as equals)
- regional ISP, connect to one or more tire-1 ISPs
- IXP (Internet Exchange point): meeting point where multiple ISPs can peer together
  <img src="https://i.loli.net/2019/03/24/5c96e8bd04c95.png" width="400px"/>

## Delay, Loss in Packet-Switched Networks

<img src="https://i.loli.net/2019/03/24/5c96eb7a50ee6.png" width="400px"/>

- Packets queue in router buffers
  - packet arrive rate to link exceeds output link capacity(packets queue, wait for turn)
- types
  - Processing delay, the time required to examine the packet's header and determine where to direct to packet
  - queuing delay, the packet waits to be transmitted onto the link, depends on congestion level of router
  - transmission delay, time to send bits into link L/R (L: packet length, R: link bandwidth), 多快的速度可以从(router)送出去
  - propagation delay, d/s = (s: propagation speed in medium, d = length of physical link)
- Transmission and Propagation Delay
    <img src="https://i.loli.net/2019/03/24/5c96edc7b4186.png" width="400px"/>
- nodal delay
  $$d_{nodal} = d_{proc} + d_{queue} + d_{trans} + d_{prop}$$

### Queueing delay

- L: packet length (bits)
- a: average packet arrival rate
- R: link bandwidth(bps)
- transfer intensity = (L*a) / R
- (L*a) / R ~ 0 : average queueing delay small
- (L*a) / R $\rightarrow$ 1: delays become large
- (L*a) /R > 1: more work arriving than can be serviced, average delay infinite, or packet loss
<img src="https://i.loli.net/2019/03/24/5c96f0920a293.png" width="400px"/>
- traceroute program: provides delay measurement from source to router along end-end internet path towards destination

### Packet Loss

- Queue(also known as buffer) preceding link in buffers has finite capacity
- when packet arrives to full queue, packet is dropped
- lost packet may be retransmitted by pervious node, by source and system, or not retransmitted at all

## Protocol Layers

- Is there any hope of organizing structure of network
- Layers: each layer implements a service
<img src="https://i.loli.net/2019/03/24/5c96f3fa35287.png" width="400px"/>
  - application: supporting network applications (FTP, SMTP, HTTP)
  - transfer: host-host data transfer (TCP, UDP)
  - network: routing of datagrams from source to destination. IP, routing protocols
  - Link: data transfer between neighboring network elements (PPP, Ethernet)
  - physical: bits "on the wire"
- why layering:
  - explicit structure allows identification, relationship of complex system's pieces
  - modularization eases maintenance, updating of system, change  of implementation of layer's service transparent to rest of system.

### Encapsulation

<img src="https://i.loli.net/2019/03/24/5c96f7453b640.png" width="450px"/>

