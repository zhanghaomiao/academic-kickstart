---
title: Application Layer
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  computer_network:
   parent: Concepts
   weight: 2
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---


Creating a network application, run on different end systems and communicate over a network, no software written for devices in network core, network core devices do not function at application layer (e.g. Web server software communicate with browser software)

## Principle of network application

### Application architectures

- Client-server
  <img src="https://i.loli.net/2019/03/24/5c9726afec632.png" width="300px"/>
  - server: always-on host, permanent IP address, server forms for scaling
  - clients: communicate with server, may be intermittently connected, may have dynamic IP address, do not communicate directly with each other
- peer-to-peer
  <img src="https://i.loli.net/2019/03/24/5c972763ee74f.png" width="300px"/>
  - no always on server
  - arbitrary end systems directly communicate
  - peers are intermittently connected and change IP address, highly scalable, but difficult to manage
- Hybrid of client-server and P2P
  - Napster, file transfer P2P, file search centralized(peer register content at central server, peers query same central server to locate content)
  - Instant message: chatting between two users is P2P, presence detection/location centralized (User registers its IP address with central server when it comes online,User contacts central server to find IP addresses of buddies)

### Processes Communicating

- process: program running within a host
- with same host, two processes communicate using inter-process communication (defined by OS)
- processes in different hosts communicated by exchanging messages (Client process: process that initiates communication, Server process: process that waits to be contacted)
- figure
  <img src="https://i.loli.net/2019/03/24/5c972a03ee1fb.png" width="450px"/>
- process sends/receives messages to/from its socket
- socket analogous to door, sending process relies on transport infrastructure on other side of door which brings message to socket at receiving process
- API (1) choice of transport protocol (2) ability to fix a few parameters
- addressing process(判断哪一个process需要沟通)
  - For a process to receive messages, it must have an identifier
  - Every host has a unique 32-bit IP address
  - Identifier includes both the IP address and port numbers(16-bit) associated with the process on the host (HTTP server: 80, Mail server: 25)

### App-layer protocol defines

- Types of messages exchanged (e.g. request & response)
- Syntax of message types (what fields in message & how field are delineated)
- Semantics of the field (i.e. meaning of information in fields)
- Rules for when and how processes send & respond to messages
- Public-domain protocols:
  - defined in RFCs
  - allows for interoperability(相容)

### What transport service does provide

- Data loss
  - some apps can tolerate some loss
  - other apps require 100% reliable data transfer
- Timing
  - some apps require low delay to be "effective"
- Bandwidth
  - some apps require minimum amount of bandwidth to be (games) effective(multimedia)
  - other apps("elastic apps") make use of whatever bandwidth they get
  <img src="https://i.loli.net/2019/03/24/5c972dff3be06.png" width="600px"/>
- TCP service: connection-oriented, reliable transport, flow control, congestion control (does not providing: timing, minimum bandwidth guarantees)
- UDP service: unreliable data transfer between sending and receiving process, dons not provide connection setup, reliability, flow control, congestion control, timing or bandwidth guarantee
<img src="https://i.loli.net/2019/03/24/5c9731b3899d9.png" width="500px"/>

## Web and HTTP

- Web
  - web page consists of objects
  - An object is a file such as HTML, file ,a JPEG image
  - A web page consists of a bast HTML-file and several referenced object
  - The base HTML file references the other objects in the page with the object's URLs (Uniform Resource Locators)

### HTTP Overview

- HTTP: hypertext transfer protocol
- web's application layer protocol
- client/server model (HTTP1.0, HTTP1.1)
- procedure
  1. client initiates TCP connection (creates socket) to server
  2. server accepts TCP connection from client
  3. HTTP message exchanged between browser and web server
  4. TCP connection closed
- HTTP is stateless, server maintains no information about post client requests

### HTTP connections

- Non-persistent HTTP
  - At most one object is sent over a TCP connection
  - HTTP/1.0 uses non-persistent HTTP
- Persistent HTTP
  - Multiple objects can be sent over singe TCP connection between client and server
  - A new connection need not be set up for the transfer of each Web object
  - HTTP/1.1 use persistent connections in default mode

#### Response time modeling (Non-persistent HTTP)

- RTT: time to send a small packet to travel from client to server and back
- Response Time: one RTT to initiate TCP connection, one RTT for HTTP request and first few bytes of HTTP response to return, file transmission time
- file transmission time = 2RTT + transmit time
<img src="https://i.loli.net/2019/03/24/5c9736941a772.png" width="500px"/>

#### Two versions of persistent connections

- Persistent without pipelining
  - client issues new request only when previous response has been received
  - One RTT for each referenced object
- Persistent with pipelining
  - default in HTTP/1.1
  - client sends requests as soon as it encounters a referenced object
  - as little as one RTT for all the referenced objects

### HTTP request message

- two types of HTTP messages: request, response
- HTTP request message (ASCII format)

```HTTP
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```

<img src="https://i.loli.net/2019/03/24/5c9739d016ad7.png" width="500px"/>

#### Method Types

- HTTP/1.0
  - GET: return the object
  - POST: send information to be stored on the server
  - Head: return only information about the object, such as how old it is, but not the object itself
- HTTP/1.1
  - GET, POST, HEAD
  - PUt: uploads a new copy of existing object in entity body to path specified in URL field
  - DELETE: delete object specified in the URL field

### HTTP response message

- **A status line**,  which indicates the success of failure of the request
- **Header line**, A description of the information in the response, this is the metadata or meta information
- **actual information**
- response status code
  - 200 OK
  - 300 Moved permanently
  - 400 Bad request
  - 404 Not found
  - 505 HTTP version not supported
<img src="https://i.loli.net/2019/03/24/5c973bc5afdcd.png" width="500px"/>

### User-Server Interaction: Cookies

- it is often desirable for a Web site to identify users
- Four components of cookie technology:
  - cookie header line in the HTTP response message
  - cookie header line in HTTP request message
  - cookie file kept on user's host and managed by user's browser
  - back-end database at web site
  <img src="https://i.loli.net/2019/03/24/5c973e402af4c.png" width="450px"/>
- advantages
  - authorization
  - shopping carts
  - recommendations
  - user session state (Web  e-mail)
- Cookies and privacy:
  - Permit sites to learn a lot about you
  - you may supply name and e-mail to sites
  - search engines use redirection & cookies to learn yet more
  - advertising companies obtain info across sites

### Web caches (proxy server)

Satisfy client request without involving origin server
<img src="https://i.loli.net/2019/03/24/5c973f6c5ec0d.png" width="400px"/>

- User sets browser: web accesses via cache
- browser sends all HTTP requests to cache
  - object in cache: cache returns object
  - else cache requests object from origin server, then returns object to client
- cache acts as both client and server
- Typically cache is installed by ISP
- why web caching
  - Reduce response for client request
  - Reduce traffic
  - Internet dense with caches enables poor content providers to effectively deliver content
- Example

### Conditional GET: client-side caching

Don't send object if client has up-to-date cached version

- client: specify date of cached copy in HTTP (`If-Modified-Since <data>`)
- server: response contains no object if cached copy is up-to-date `HTTP/1.1 304 Not Modified`

## FTP (File transfer protocol)

<img src="https://i.loli.net/2019/03/24/5c9744679515d.png" width="400px"/>

- transfer file to/from remote host
- client/server model
  - client side: the side that initiates transfer
  - server side: remote host
- ftp : RFC 959
- ftp server port 21
- Procedure
  1. FTP client contacts FTP server at port 21, specifying TCP as transport protocol
  2. Client obtains authorization over control connection
  3. Client browses remote directory by sending commands over control connection
  4. When server receives a command for a file transfer, the server opens a  TCP data connection to client
  5. After transferring one file, server closes connection
  6. Servers open a second TCP data connection to transfer another file
- FTP server maintains "state": current directory

### FTP Commands, responses

- commands
  - USER, username
  - PASS, password
  - LIST, return list of file in current directory
  - RETR filename -- retrieves (gets) file
  - STOR filename -- stores file onto remote host
- response
  - 331 Username OK, password required
  - 125 data connection already open, transfer starting
  - 425 can't open data connection
  - 452 Error writing file

## Electronic Mail (SMTP, POP3, IMAP)

Three major components of a mail system user agents, mail servers ,simple mail transfer protocol: SMTP
  <img src="https://i.loli.net/2019/03/24/5c974714a2626.png" width="450px"/>

- User Agent:
  - known as "mail reader"
  - composing, editing, reading mail messages
  - outgoing, incoming message stored on server
- Mail server:
  - mailbox: contains incoming messages for user
  - message queue: of outgoing (to be sent) mail messages
- protocol

### SMTP protocol

- client: sending mail receiver
- server: receiving mail server
- use TCP to reliably transfer email message from client to server, port25
- direct transfer: sending server to receiving server
- three phases of transfer
  1. handshaking
  2. transfer of messages
  3. closure
- command/response interaction
  - command: ASCII text
  - response: status code and phrase
- SMTP use persistent connections
- SMTP requires message to be in 7-bit ASCII
- SMTP server uses `CRLF.CRLT` to determine end of message （`CRLF` 表示换行）
<img src="https://i.loli.net/2019/03/24/5c9748876c1b9.png" width="500px"/>
- comparison with HTTP
  - HTTP pull protocol (client's point of view), SMTP(push protocol)
  - both have ASCII command/response interaction, status codes
  - HTTP dons not require message to be in 7-bit ASCII
  - HTTP: one object in  one response message
  - SMTP: multiple objects can be set in one message

### Message format

- From, To, Subject
- MIME(Multipurpose Internet Mail extensions)
- additional lines in message header declare MIME content type
- MIME types: (TEXT, Image, Audio, Video, Application, Multipart, Message)

### Mail access protocols

- SMTP: delivery/storage to receiver's server
- Mail access protocol: retrieval from server
  - POP3: Post Office Protocol, version 3 (authorization (agent -> server) and download)
  - IMAP: Internet Mail Access Protocol (RFC 2060)
    - more features (more complex)
    - manipulation of stores messages on server
  - HTTP: Hotmail, Gmail etc
  <img src="https://i.loli.net/2019/03/24/5c9764e842cbf.png" width="500px"/>

#### POP3 protocol

1. client opens a TCP connection to the mail server on port 110
2. authorization phase (client: declare username, password, server response: +OK)
3. transaction phase: client
   - `list`: list message number
   - `retr`: retrieve message by number
   - `dele`
   - `quit`
4. update phase: mail server deletes the message marked for deletion

- **download and delete**
- **download and keep**: copies of messages on different clients
- POP3 is stateless across sessions

#### IMAP

- keep all messages in one place: the server
- Allows user to organize messages in folders
- IMAP keeps user state across sessions (names of folders and mappings between message IDs and folder name)

## DNS (domain name system)

MAP between IP addresses and name

- A distributed database implemented in hierarchy of many name servers
- no server has all name-to-IP address mappings
- Ap application-layer protocol that allows host, routers, name servers to communicate to resolve names(address/name translation)
  - DNS provides a core Internet function, implemented as application-layer protocol
  - DNS is an example of the Internet design philosophy of placing complexity at network's edge
- Services
  - Mapping
  - Host aliasing (Canonical and alias name)
  - Mail server aliasing
  - Load distribution, Replicated Web servers: set of IP addresses for one canonical name (负载平衡)
- Structures
<img src="https://i.loli.net/2019/03/24/5c9769b66f470.png" width="400px"/>
- Client wants IP for *www.amazon.com*
  - client queries a root server to find com DNS server
  - client queries com DNS server to get amazon.com DNS server
  - Client queries amazon.com DNS server to get IP address for www.amazon.com
- Four types of name services
  1. root name servers
  2. top level name servers
  3. authoritative name servers
      - for a host: stores that host's IP address, name
  4. local name servers, each ISP, company has local(default) name server, host DNS query first goes to local name server
- Top-level domain(TLD) servers: responsible for com, org, net, edu, etc, and all top-level country domains uk, fr, ca, jp
  - The company **Network solutions** maintains servers for com TLD
  - The company **Educause** maintains servers for edu TLD
- Authoritative DNS servers: organization's DNS servers, providing authoritative hostname to IP mappings for organization's servers, can be maintained by organization or service provider
- local name server: does not strictly belong to hierarchy, each ISP has one
- Example (iterative query)
  <img src="https://i.loli.net/2019/03/24/5c976d5a0de29.png" width="400px"/>
- recursive query (puts burden of name resolution on contacted name server)
  <img src="https://i.loli.net/2019/03/24/5c976e1ace932.png" width="400px"/>
- caching and updating
  - once name server learns mapping, it caches mapping, caches entries timeout(disappear) after some time
  - the contents of each DNS servers were configured **statically** from a configuration file created by a system manager
  - An update option has been added to the DNS protocol to allow data to be added or deleted from the database vid DNS messages
- DNS records
  distributed database storing resource records (RR), RR format: (name, value, type, ttl)

### DNS protocol, messages

- DNS protocol query and reply messages, both with same message format
<img src="https://i.loli.net/2019/03/24/5c97709813988.png" width="400px"/>

## Peer-to-Peer Applications

File distribution problem (client-server model)
<img src="https://i.loli.net/2019/03/24/5c977afd15e0a.png" width="400px"/>
<img src="https://i.loli.net/2019/03/24/5c977b81b3242.png" width="400px"/>

### Bit Torrent

<img src="https://i.loli.net/2019/03/24/5c977bf370a09.png" width="400px"/>

### Structure

- P2P centralized directory
  - when peer connects, it informs central server (IP address, content)
  - Single point of failure
  - performance bottleneck
  - copyright infringement

## Socket Programming

build client/server application communication using sockets

- client/server paradigm
- types of transport service via socket API (unreliable datagram, reliable, byte stream-oriented)
- socket: a host-local, application-created, OS-controlled interface("door") into which application can both send and receive message to/from another application process
- Socket Programming using Java, cross platform without recompiling, easy programming with high-level API

### Socket-programming using TCP

- Socket: a door between application process and end-to-end transport protocol (UCP or TCP)
- TCP service: reliable transfer of bytes from one process to another

1. Client must contact server
    - server process must first be running
    - server must have created socket (door) that welcomes client's contact
    - client contacts server by
      - creating client-local TCP socket
      - specifying IP address, port number of server process
      - when client creates socket: client TCP established connection to server TCP
2. when contacted by client, server TCP **creates new socket** for server process to communicate with client
    - allows server to talk with multiple clients
    - source port numbers used to distinguish clients

<div class="info note"><p>
Stream
<ul>
<li>A stream is a sequence of characters that flow into or out of a process</li>
<li>An input stream is attached to some input source for the process (e.g. keyboard or socket)</li>
<li>An output stream is attached  to an output source (e.g. monitor or socket)</li>
</ul>
 <p></div>

<img src="https://i.loli.net/2019/03/26/5c999a9e7389e.png" width="500px"/>

### TCP client

- A "Socket" object, making connection requests
- Parameters
  - Remote ip address
  - Remote tcp port
  - Local ip address (optional)
  - Local port (optional)
- Example

```java

import java.net.*;
import java.io.*;

public class Client{
    private Socket socket = null;
    private DataOutputStream out = null;
    private BufferedReader input;

    public Client(String address, int port)
    {
        try {
            socket = new Socket(address, port);
            System.out.println("Connected");
            input = new BufferedReader(new InputStreamReader(System.in));
            out = new DataOutputStream(socket.getOutputStream());
        }
        catch (UnknownHostException u){
            System.out.println(u);}
        catch (IOException i){
            System.out.println(i);}

        String line = "";
        while(!line.equals("Over"))
        {
            try{
                line = input.readLine();
                out.writeUTF(line);
            }
            catch(IOException i)
            {
                System.out.println(i);
            }
        }
        try{
            input.close();
            out.close();
            socket.close();
        }
        catch(IOException i)
        {
            System.out.println(i);
        }
    }
    public static void main(String[] args) {
        Client client = new Client("127.0.0.1", 5000);
    }
}
```

```java
import java.net.*;
import java.io.*;

public class Server{
    private Socket socket = null;
    private ServerSocket server = null;
    private DataInputStream in = null;

    public  Server (int port){
        try{
            server = new ServerSocket(port);
            System.out.println("Server started");
            socket = server.accept();
            System.out.println("client accepted");

            in = new DataInputStream(socket.getInputStream());

            String line =  "";
            while (!line.equals("Over"))
            {
                try{
                    line = in.readUTF();
                    System.out.println(line);
                }
                catch (IOException i)
                {
                    System.out.println(i);
                }
            }
            System.out.println("Closing connected");
            socket.close();
            in.close();
        }
        catch (IOException i){
            System.out.println(i);
        }
    }
    public static void main(String[] args){
        Server server = new Server(5000);
    }
}
```

### Java socket Programming With thread

```java
import java.io.*;
import java.text.*;
import java.util.*;
import java.net.*;

public class Server {
    public static void main(String[] args) throws IOException
    {
        ServerSocket ss = new ServerSocket(5056);
        while (true)
        {
            Socket s = null;
            try{
                s = ss.accept();
                System.out.println("A new client is connected");
                DataInputStream dis = new DataInputStream(s.getInputStream());
                DataOutputStream dos = new DataOutputStream(s.getOutputStream());
                System.out.println("Assign new thread for this client");
                Thread t = new ClientHandler(s, dis, dos);
                t.start();
            }
            catch (Exception e){
                s.close();
                e.printStackTrace();
            }
        }
    }
}

class ClientHandler extends Thread{
    private DateFormat fordate = new SimpleDateFormat("yyyy/MM/dd");
    private DateFormat fortime = new SimpleDateFormat("hh:mm:ss");
    final private DataInputStream dis;
    final private DataOutputStream dos;
    final private Socket s;

    ClientHandler(Socket s, DataInputStream dis, DataOutputStream dos){
        this.dis = dis;
        this.dos = dos;
        this.s = s;
    }

    @Override
    public void run(){
        String received;
        String toreturn;
        while(true){
            try{
                dos.writeUTF("what do you want? [Date|TIme]..\n" + "Type exit to terminate connection");
                received = dis.readUTF();
                if(received.equals("Exit"))
                {
                    System.out.println("Client" + this.s + "Sends exist");
                    System.out.println("Closing this connection");
                    this.s.close();
                    System.out.println("Connection closed");
                    break;
                }
                Date date = new Date();
                switch (received) {
                    case "Date":
                        toreturn = fordate.format(date);
                        dos.writeUTF(toreturn);
                        break;
                    case "Time":
                        toreturn = fortime.format(date);
                        dos.writeUTF(toreturn);
                        break;
                    default:
                        dos.writeUTF("Inavalid input");
                        break;
                }
            }
            catch (IOException e)
            {
                e.printStackTrace();
            }
        }
        try {
            this.dis.close();
            this.dos.close();
        } catch(IOException e)
        {
            e.printStackTrace();
        }
    }
}
```

```java
import java.io.*;
import java.net.*;
import java.util.Scanner;

public class Client {
    public static void main(String[] args)
    {
        try {
            Scanner scn = new Scanner(System.in);
            InetAddress ip = InetAddress.getByName("localhost");
            Socket s = new Socket(ip, 5056);
            DataInputStream dis = new DataInputStream(s.getInputStream());
            DataOutputStream dos = new DataOutputStream(s.getOutputStream());

            while (true) {
                System.out.println(dis.readUTF());
                String tosend = scn.nextLine();
                dos.writeUTF(tosend);

                if (tosend.equals("Exit")) {
                    System.out.println("Closing the connection");
                    s.close();
                    System.out.println("Connection closed");
                    break;
                }
                String received = dis.readUTF();
                System.out.println(received);
            }
            scn.close();
            dis.close();
            dos.close();
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
    }
}
```

### Socket programming with UDP

- no handshaking
- sender explicitly attaches IP address and port of destination to each packet
- server must extract IP address, port of sender from received packet
- transmitted data may be received out of water or lost
<img src="https://i.loli.net/2019/03/29/5c9dc3a0bc9fa.png" width="400px"/>
- Exception Handling during initializing sockets, establishing connection, data transmission