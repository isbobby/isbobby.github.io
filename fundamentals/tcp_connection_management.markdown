---
title: TCP Connection Management
parent: Fundamentals
nav_order: 1
---
# TCP Connection Management
References:
- [Microchip's introduction to TCP/IP](https://developerhelp.microchip.com/xwiki/bin/view/applications/tcp-ip/sockets-ports/)
- [TCP Guide](http://www.tcpipguide.com/free/t_TCPConnectionTermination-2.htm)
- [Free BSD Guide](https://man.freebsd.org/cgi/man.cgi?query=listen&sektion=2&format=html)

# Key Concepts
Ports and sockets are important concepts to TCP connection management.

## Ports
These integer ports are used to identify the application Layer processes running on the host.

This is necessary as an IP address helps to identify the machine on a network, and the port helps to identify the process on a machine. Some of the ports are reserved / well known.

In a request / response model, we usually have one server to serve many clients. The server's port is fixed. We also need to identify the client process with a port. However, this port can be generated and assigned by the transport layer itself. These ports are allocated for short term use, and are referred to as ephemeral ports
## Sockets
Transport Layer interfaces with the Application Layer, **socket** is the software concept used for this interface.

An application needs to create a socket to connect to an application on another host, messages are transmitted through this socket.

![](micro_chip_socket_image.png)

Image Credit : [Microchip](https://developerhelp.microchip.com/xwiki/bin/view/applications/tcp-ip/sockets-ports/#HTCP2FIPPorts)

Internally, sockets implement receive and transmit buffers - **TX** & **RX** buffers. When an application wants to transmit a packet, its process writes the packet to the socket's **TX** buffer. Similarly, data is read from the **RX** buffer.

## **Socket Operations**
Application needs to manage the life cycle of a socket. For TCP, the following step applies
1. server creates a socket, checks for incoming messages
2. client creates a socket connecting to the server, server assigns new socket for this connection
3. data transmission using buffers
4. sockets on both server and clients are closed

Sockets are also embedded with states, these states allow processes to perform the right system calls to transit from one state to the next.

### **1 - Server socket creation**
A server needs to create socket to receive incoming connections from client, via the following steps
1. creating the socket, with `sockfd = socket()`, getting the file descriptor of the socket. The state of the socket now is `CLOSED`
2. set the option of the socket, with `setsockopt`
3. bind the socket to the address and port number in `bind(sockfd, addr, addr_len)`
4. puts the socket into the passive mode with `listen(sockfd, backlog)` with a backlog queue ([BSD ref](https://man.freebsd.org/cgi/man.cgi?query=listen&sektion=2&format=html)), the socket transits to `LISTEN` state.
5. listens and creates new socket with `new_socket = accept(sockfd, addr, addr_len)` later on

After `listen()`, this socket is a passive socket, which may not initiate connection. It also has a queue to hold pending connections.

### **2 - Client socket creation & establishing TCP connection**

A client should be aware of the destination address and port. Before sending data to the server, it needs to create a socket with the destination address, with
1. creating a new socket, same as server, with `sockfd = socket()`, the state of the client socket now is `CLOSED`
2. connect socket to server with `connect(sockfd, server_addr, server_addr_len)`, this attempts to establish a TCP connection.

When `connect()` is called, the kernel performs TCP three way handshake, sockets on both server and client begin transiting through different states.

**`SYN`**: At this stage, the client socket transitions from `CLOSED` to the `SYNC_SENT` stage, and after the server receives the `SYN` successfully, it allocates a new socket, and transitions the new socket to the `SYN_RCVD` state.

**`SYN-ACK`**: Next, the server writes the `SYN-ACK` packet to the socket, the client waits for this packet and upon receiving, transitions to the `ESTABLISHED` state and respond to the server with the final `ACK`.

**`ACK`**: Lastly, upon receiving the `ACK`, the server also transitions to the `ESTABLISHED` state. Both the server and client sockets are fully established.

### **3 - Data transmission via socket buffers**

After both sockets are created, client can attempt delivery. The process will write data to the **TX** buffer of its socket, with `write(fd, buffer, buffer_size)`, and read using `read()`.

### **4 - Terminating connection**

Only client process can establish a connection with the first `SYN` packet, but either client or server can initialise connection termination with the first `FIN` packet.

Since there could be data remaining in the buffers when one side sends the `FIN` request, it may take sometime to complete the delivery and send its own `FIN`. Therefore, termination handshake does not group `FIN` and `ACK` into one like `SYN-ACK`.

After sometime, the other process can send its own `FIN` to finally indicate it's not sending more data. Connection is terminated, and the file descriptor, port and resources are released back to the OS.

![](tcp_4_way_handshake.png)

### **Berkeley / BSD**

The Berkeley sockets are a set of APIs defined for network communication, this set of APIs have become the de facto standard for socket based networking.

Some key functions include
`socket()` - creating a socket and returns
`bind()` - associate a socket with a local port
`listen()` - marking the socket as **passive**, allows it to accept for incoming connections
`connect()` - used by client to initiate a connection
`send() / write()` and `read() / recv()` - socket IO
`close()` - closes a socket and release resources.
`select() / poll()` - monitors multiple sockets to see if they are ready for reading

# Implementations
[(WIP) Implement TCP client and server in C](https://github.com/isbobby/sockets-programming/tree/main/c)