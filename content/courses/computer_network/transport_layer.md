---
title: Transport Layer
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  computer_network:
   parent: Concepts
   weight: 3
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 2
---

## Transport services and protocols

- provide logical communication between app processes running on different hosts
- transport protocols run in end systems
  - send side: breaks app message into segments, passes to network layer
  - receive side: reassembles segments into message passes to app layer
- more than one transport protocol available to apps (Internet: TCP and UDP)
<img src="https://i.loli.net/2019/03/29/5c9dd08c3e25b.png" width="400px"/>
- network layer vs. transport layer
  - network layer: logical communication between hosts
  - transport layer: logical communication between processes
- reliable, in-order delivery(TCP), congestion control, flow control, connection setup
- unreliable, unordered delivery(UDP): no-frills extension of "best-effort" IP
- services not available: delay guarantees, bandwidth guarantees

## Multiplexing / demultiplexing

<img src="https://i.loli.net/2019/03/29/5c9dd2c317a10.png" width="400px"/>

- demultiplexing: delivering received segments to correct socket
- multiplexing: gathering data from multiple sockets, enveloping data with header(later used for demultiplexing)

### Demultiplexing

- Host receive IP datagrams:
  - each datagram has source IP address, destination IP address
  - each datagram carries 1 transport-layer segment
  - each segment has source, destination port number
- host uses IP addresses & port numbers to direct segment to appropriate socket
<img src="https://i.loli.net/2019/03/29/5c9dd47a8dd0d.png" width="400px"/>

#### Connectionless demultiplexing

- create sockets with port number
- UDP socket identified by two-tuple(destination IP address and destination port number)
- when host receives UDP segment: checks destination port number in segment, directs UDP segment to socket with that port number
- IP datagrams with different source IP addresses and/or source port numbers directed to same socket

#### Connection-oriented demultiplexing

- TCP socket identified by 4-tuple: source/destination IP address, source/destination port number
- receiving host uses all four segment to appropriate socket
- Server host may support many simultaneous TCP sockets, each socket identified by its own 4-tuple
- Web servers have different sockets for each connection client, non-persistent HTTP will have different socket for each request
<img src="https://i.loli.net/2019/03/29/5c9dda9a2fa8b.png" width="400px"/>

## UDP: User Datagram Protocol (RFC 768)

- "no frills", "bare bones" Internet transport protocol
- "best effort" service
- connectionless, no handshaking between UDP sender, receiver, each UDP segment handled independently of others
- why using UDP:
  - no connection establishment
  - simple: no connection state at sender, receiver
  - small segment header
  - no connection control: UDP can blast away as fast as desired
- often used for streaming multimedia apps
- other UDP uses (DNS, SNMP(Simple network manager protocol) 网络管理资料收集)
- reliable transfer over UDP: add reliability at application layer(application-specific error recovery)
<img src="https://i.loli.net/2019/03/29/5c9ddcd48373c.png" width="400px"/>

### UDP checksum

- Goal: detect "errors" in transmitted segment
- sender:
  - treat segment contents as sequence of 16-bit integers
  - checksum: addition (1's complement sum) of segment contents
  - sender puts checksum value into UDP checksum fields
- Receiver:
  - compute checksum of received segment
  - check if computed checksum equals checksum field value
(No, error-detected, Yes, no error-detected)

## Principles of Reliable data transfer

- top-10 list of important networking topics
<img src="https://i.loli.net/2019/03/29/5c9ddf4ae2bd3.png" width="600px"/>
- characteristics of unreliable channel will determine complexity of reliable data transfer protocol
- `rdt_send()`: called from above, passed data to deliver to receiver upper layer
- `udt_send()`: called by `rdt`, to transfer packet over unreliable channel to receiver
- `rdt_rcv()`: called when packet arrives on `rcv-side` of channel
- `deliver_data()` called by `rdt` to deliver data to upper

### Reliable data transfer

- incrementally develop sender, receiver sides of reliable data transfer protocol(rdt)
- consider only unidirectional data transfer(but control info will flow on both directions)
- use finite state machines(FSM) to specify sender, receiver

### Rdt1.0: reliable transfer over a reliable channel

- underlying channel perfectly reliable
  - no bit errors
  - no loss of packets
- separate FSMs for sender, receiver
  - sender sends data into underlying channel
  - receiver read data from underlying channel
<img src="https://i.loli.net/2019/03/29/5c9df830253ec.png" width="600px"/>

### Rdt2.0: channel with bit errors

- underlying channel may flip bits in pocket: checksum to detect bit errors
- how to recover from errors:
  - acknowledgements(ACKs) receiver explicitly tells sender that packet received OK
  - negative acknowledgements(NAKs) receiver explicitly tells sender that packet had errors
  - sender retransmits packet on receipt of NAK
- new mechanisms in rdt2.0
  - error detection
  - receiver feedback: control msgs(ACK, NAK)
  <img src="https://i.loli.net/2019/03/29/5c9dfa762b88f.png" width="600px"/>
- rdt2.0 has a fatal flow(if ACK/NAK corrupted?)
  - sender doesn't know what happened at receiver
  - can't just retransmit: possible duplicate
- Handling duplicates:
  - sender retransmits current packet if ACK/NAK corrupted
  - sender adds sequence number to each packet
  - receiver discards (doesn't deliver up) duplicate packet
- Stop and wait (sender sends one packet, then waits for receiver response)

### Rdt2.1: sender, handles garbled ACK/NAKs

- Sender
<img src="https://i.loli.net/2019/03/29/5c9dfcac45e5e.png" width="600px"/>
- Receiver
<img src="https://i.loli.net/2019/03/29/5c9dfd8b78c4b.png" width="600px"/>

### Rdt2.2: a NAK-free protocol

- same functionality as rdt2.1, using ACKS only
- instead of NAK, receiver sends ACK for **last packet** received OK, receiver must explicitly include sequence number of packet being ACKed
- duplicate ACK at sender results in same action as NAK: retransmit current packet

### Rdt3.0: channels with errors and loss

- New assumption:
  - underlying channel can also pockets(data or ACKs)
  - checksum, sequence number, ACKs, retransmission will be of help, but not enough
- Approach: sender waits "reasonable" amount of time for ACK
- retransmits if no ACK received in this time
- if packet (or ACK) just delayed (not lost):
  - retransmission will be duplicate, but use of sequence number
  - receiver must specify sequence number of packet being ACKed
- requires countdown timer
<img src="https://i.loli.net/2019/03/29/5c9e04ebd9810.png" width="600px"/>
- performance of rdt3.0
  1Gbps link, 15 ms prop. delay, 8000 bit packet:
  - sender time
    $$ d = L/R = \frac{8000 bits}{10^9bps} = 8 ms$$
  - utilization - fraction of time sender busy sending
    $$ \frac{L/R}{RTT + L/R} = 0.008/30.0008 = 0.00027$$
- stop and wait operation
  <img src="https://i.loli.net/2019/03/29/5c9e07305c4bb.png" width="500px"/>

### Pipelined protocols

- sender allows multiple, "in-flight" yet-to-be-acknowledged` packets
  - range of sequence numbers must be increased
  - buffering at sender and/or receiver
<img src="https://i.loli.net/2019/03/29/5c9e07a57c05d.png"/>
- Two generic forms of pipelined protocols: **go-Back-N, selective repeat**

#### Go-back-N: big picture

- sender can have up to N unacked packets in pipeline
- Receiver only sends cumulative ACKs, doesn't ACK packet if there's gap
- Sender has timer for oldest unacked packet, if timer expires, retransmit all unlocked packets
<img src="https://i.loli.net/2019/03/29/5c9e09de9caf2.png"/>
- ACK(n): ACKs all packet up to including sequence number n -- "cumulative ACK" (may receive duplicate ACKs)
- timer for each in-flight packet
- timeout(n): retransmit packet n and all higher sequence number packets in window
<img src="https://i.loli.net/2019/03/29/5c9e0ef5f0e00.png" width="600px"/>
- sender
  <img src="https://i.loli.net/2019/03/29/5c9e0ef5f0e00.png" width="600px"/>
- receiver
  <img src="https://i.loli.net/2019/03/31/5ca036ac9a793.png" width="600px"/>  
- example
  <img src="https://i.loli.net/2019/03/31/5ca036ee8e6d8.png" width="400px"/>

#### Selective Repeat: big picture

- Sender can have up to N unacked packets in pipeline
- Receiver ACKs individual packets
- Sender maintains timer for each unacked packet, when timer expires, retransmit only unacked packet
<img src="https://i.loli.net/2019/03/31/5ca0371c00c83.png" width="400px"/>
- Sender
  - if next available sequence number in window, send packet
  - timeout(n): resend packet n, restart timer
  - ACK(n) in (sendbase, sendbase + N): mark packet n as reader, if n smallest unACked packet, advance window base to next unACKed sequence number
- receiver
  - send ACK(N)
  - out-of-order : buffer
  - in-order: deliver, advance window to next not-yet-received packet
  - if packet n in [revbase-revbase+N], ACK(n), otherwise ignore
<img src="https://i.loli.net/2019/03/31/5ca03a0f20daa.png" width="400px"/>
- problem
  <img src="https://i.loli.net/2019/03/31/5ca03dc05ef79.png" width="500px"/>
  window size must be less than or equal t0 half the size of the sequence number space

## Connection-Oriented Transport: TCP

### Overview

- Point-to-Point
- reliable: in-order byte stream: no "message boundaries"
- pipelined: TCP congestion and flow control set window size
- send & receive buffer
- flow control: sender will not overwhelm receiver
- full duplex data: bi-directional data flow in some connection, maximum segment size(MSS， 控制流量，控制传输速度)
- connection-oriented: handshaking (Exchange of control messages)
- TCP segment structure
<img src="https://i.loli.net/2019/04/01/5ca1b0f974767.png" width="400px"/>
- Sequence number: byte stream "number" of first byte in segment's data
- ACKs: sequence number of next byte expected from other side, cumulative ACK
<img src="https://i.loli.net/2019/04/01/5ca1b4dc5a0aa.png" width="400px"/>

### Round-Trip Time Estimation and Timeout

- how receiver handles out-of-order segments: TCP spec doesn't say, up to implementor
- how to set TCP timeout value?
  - longer than RTT(Round Trip Time)
  - too short: premature timeout (unnecessary retransmission)
  - too long: slow reaction to segment loss
- how to estimate RTT?
  - SampleRTT: measured time from segment transmission until ACT receipt (ignore retransmissions)
  - SampleRTT will vary, want estimated RTT "smoother", average several recent measurement, not just current SampleRTT
- EstimatedRTT = $(1-\alpha)\cdot \text{EstimatedRTT} + \alpha \cdot \text{SampleRTT} \text{ typically}, \alpha$ is 0.125
- estimate of how much SampleRTT deviates from EstimatedRTT
  $DevRTT = (1-\beta) \cdot \text{DevRTT} \cdot \beta \cdot |\text{SampleRTT}-\text{Estimated RTT}|$, typically, $\beta = 0.25$
- set timeout interval
  TimeoutInterval = EstimatedRTT + 4 * DevRTT

### TCP sender events

- data received from application:
  - create segment with sequence number
  - sequence number is a byte-stream number of first data byte in segment
  - start timer if not already running (think of timer as for oldest unacked segment)
  - expiration interval: timeout Interval
- timeout: retransmit segment that caused timeout, restart timer
- ACK received: if acknowledges previously unacked segments, update what is known to be acked, start timer if there are outstanding segments
- figure:
<img src="https://i.loli.net/2019/04/01/5ca1febf72143.png" width="700px"/>
- TCP retransmission Scenarios
<img src="https://i.loli.net/2019/04/01/5ca1ff1ec67a4.png" width="400px"/>
<img src="https://i.loli.net/2019/04/01/5ca1ff540a9f6.png" width="450px"/>
<img src="https://i.loli.net/2019/04/01/5ca1ff7dbea07.png" width="4550px"/>

### TCP ARK Generation

<img src="https://i.loli.net/2019/04/01/5ca1ffd31e346.png" width="600px"/>

### Fast retransmit

- Time-out period often relatively long (long delay before resending lost packet)
- detect lost segment via duplicate ACKs (If segment is lost, there will likely be many duplicate ACKs)
- if Sender receives ACKs for the same data, it supposes that segment after ACKed data was lost
- fast retransmit: resend segment before timer expires
<img src="https://i.loli.net/2019/04/01/5ca200cc1f243.png" width="450px"/>
- Algorithm
<img src="https://i.loli.net/2019/04/01/5ca20110e65d0.png" width="600px"/>

### Flow Control

- Sender won't overflow receiver's buffer by transmitting too much or too fast
- Receive side of TCP connection has a receive buffer
<img src="https://i.loli.net/2019/04/01/5ca2019f72288.png" width="400px"/>
- Suppose TCP receiver discards out-of-order segments
- Receiver advertises spare room by including value of  RcvWindow in segments
- Sender limits unACKed data to RcvWindow
- rwnd(receive window) = RcvBuffer - [LastByteRcvd - LastByteRead]

### TCP Connection Management

- Three way handshake
  1. Client host sends TCP SYN segment to server, specifies initial sequence number (no data)
  2. Server host receivers SYN, replies with SYNACK segment, server allocates buffers, specifies server initial sequence number
  3. client receives SYNACK, replies with ACK segment, which may contain data
  <img src="https://i.loli.net/2019/04/01/5ca205d6c7de8.png" width="400px"/>
- Closing a connection
  1. client end system send TCP FIN control segment to server
  2. server receives FIN, replies with ACK, closes connection, sends FIN
  3. client receives FIN, replies with ACK, enters "timed wait" will respond with ACK to received FINs
  4. server, receives ACK, connection closed
  <img src="https://i.loli.net/2019/04/01/5ca205f6dfd5c.png" width="400px"/>
- Figure
<img src="https://i.loli.net/2019/04/01/5ca206338a961.png" width="400px"/>
<img src="https://i.loli.net/2019/04/01/5ca2066d19100.png" width="400px"/>

### TCP congestion control

#### Principles of Congestion Control

- too many sources sending too much data too fast for network to handle
- manifestations: lost packets, long delays
- a top-10 problem
- Cause/costs of congestion: scenario 1, two connections sharing a single hop with infinite buffers, maximum achievable throughput, large delays when congested
<img src="https://i.loli.net/2019/04/01/5ca20729465a5.png" width="450px"/>
<img src="https://i.loli.net/2019/04/01/5ca2079000572.png" width="500px"/>
- scenario 2, one router, finite buffers, and retransmission
<img src="https://i.loli.net/2019/04/01/5ca207dfa7b04.png" width="400px"/>
<img src="https://i.loli.net/2019/04/01/5ca2083286448.png" width="500px"/>
  - always: $\lambda_{in} = \lambda_{out}$
  - "perfect" retransmission only when loss: $\lambda_{in}' = \lambda_{out}$
  - retransmission of delayed (not lost) packet makes $\lambda_{in}'$ larger (than perfect case) for same $\lambda_{out}$
- Scenario 3, four senders, multi-hop paths, timeout/retransmit
<img src="https://i.loli.net/2019/04/01/5ca2092db9321.png" width="400px"/>
<img src="https://i.loli.net/2019/04/01/5ca209691eb47.png" width="400px"/>
  - even more crowded
  - when packet dropped, any upstream transmission capacity used for that packet was wasted (之前走过的路径全部浪费掉)

#### Approaches towards congestion control

- End-end congestion control
  - no explicit feedback from network
  - congestion inferred from end-system observed loss, delay
  - approach taken by TCP
- Network-assisted congestion control:
  - routers provide feedback to end system
  - single bit indicating congestion (SNA, TCP/IP ECN, ATM)
  - explicit rate sender should send
- CASE: ATM (ABR congestion control)
  - ABR: available bit rate, elastic service
    - if sender's path "underloaded", sender should use available bandwidth
    - if sender's path congested: sender throttled to minimum guaranteed rate
  - RM (resource management) cells:
    - sent by sender, interspersed with data cells
    - bit in RM cell set by switches: NI bit: no increase in rate; CI bit: congestion indication
    - RM cells returned to sender by receiver, with bits intact
  <img src="https://i.loli.net/2019/04/01/5ca20b6a0066b.png" width="400px"/>
  - EFCI bit in data cells: set to 1 in congested switch, if data cell preceding RM cell has EFCI set, receiver sets CI bit in returned RM cell

#### TCP congestion control

- Congestion window(cwnd): imposes a constraint on the rate at which a TCP sender can send traffic into the network,
LastByteSent – LastByteAcked <= cwnd
- MSS: maximum segment size: the maximum amount of data that can be grabbed and placed in a segment
- roughly rate = CongWin / RTT (Bytes/sec)
- How does sender perceive congestion?
  - loss event = timeout or 3 duplicate ACKs
  - TCP sender reduces rate (cwnd) after loss event
- three mechanisms:
  - AIMD
  - slow start
  - conservative after timeout events

##### AIMD

- Approach: increase transmission rate (window size), probing for usable bandwidth, until loss occurs
- Additive increase: increase CongWin by 1 MSS every RTT until loss detected
- multiplicative decrease: cut CongWin in half after loss
<img src="https://i.loli.net/2019/04/01/5ca20c3c5c2a0.png" width="400px"/>

##### Slow start

- when connections begins, CongWIn = 1MSS
- available bandwidth may be >> MSS/RTT
- when connection begins, increase rate exponentially fast until first loss event
  - double congwin every RTT
  - done by incrementing congwin for every ACK received
<img src="https://i.loli.net/2019/04/01/5ca20eb32840b.png" width="400px"/>
- inferring loss
  - After 3 duplicate ACKs, cwnd is cut in half, window the grows linearly
  - after timeout event, cwnd set to 1 MSS, window then grows exponentially to a threshold, then grows linearly
  <img src="https://i.loli.net/2019/04/01/5ca20f8ca48d5.png" width="400px"/>

#### TCP Summary

- TCP throughput: Let W be the window size when loss occurs, when window is W, throughput is W/RTT, just after loss, windows drops to W/2, throughput is W/(2RTT), average throughput 0.75W/RTT
- TCP future: TCP over "long, fat pipes"
  1500 byte segments, 100ms RTT, want 10Gbps throughputs, requires window size W = 83.333 in flight ($W \cdot 1500 \cdot 8 / 100ms = 10Gbps$), throughput in terms of loss rate: $1.22\cdot MSS/(RTT\sqrt{L})$ , $L = 2\cdot 10^{-10}$
- TCP Fairness, if K TCP sessions share same bottleneck link of bandwidth R, each should have average rate of R/K, the TCP is fair
<img src="https://i.loli.net/2019/04/01/5ca2208dd5029.png" width="400px"/>