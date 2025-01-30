---
title: TCP Connection Management
parent: Computer Network
nav_order: 2
---
# Outline
The basic TCP server program should do the following

1. create a TCP socket
2. optinally configure the socket 
3. binding created socket to an address
4. transform the socket into a passive socket 
5. accept incoming connection and handle requests
6. close connection and clean up resources

## Creating a TCP Socket
We need to refer to [man `socket(2)`](https://man7.org/linux/man-pages/man2/socket.2.html), where we obtain the following key information
```c
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
```

Following the doc, we will use `AF_INET` as the `domain` argument, and `SOCK_STREAM` as the `type` argument. Digging down further, we see this quote

>  The protocol specifies a particular protocol to be used with the socket. Normally only a single protocol exists to support a particular socket type within a given protocol family, in which case protocol can be specified as 0

Since `SOCK_STREAM` is implemented on TCP only, we can leave it as 0. `man socket(2)` also provides a link to `man protocols(5)`, which lists the protocol in `etc/protocols`.

If we do `cat /etc/protocols | grep TCP`, we get the following (on my Macbook).
```
tcp	6	TCP		# transmission control protocol
```

With the above information, we can write the following code (note, the `close()` is added here but will be discussed later). We can use `0` as protocol, but `6 (tcp)` will be functionally the same.
```c
#include <sys/socket.h>
#include <unistd.h>

int main() {
    int sockfd;
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    close(sockfd);
}
```
## Binding socket to an address
The created socket now does not have address assigned to it yet. `bind()` assigns the address specified by `addr` to the socket. 

```c
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

Based on `man bind(2)`, the `struct sockaddr` is to cast the incoming structure pointer to avoid compiler warning, the actual structure depends on the address family. In `man sockaddr`, we found ` sockaddr_in` which is suitable for IPv4.

We rely on `#include <arpa/inet.h>` to provide `inet_addr` to convert a stirng to usable address, and `htons` (host to network) to convert `8080` to the right endianness.

```c
#include <netinet/in.h>
#include <arpa/inet.h>

struct sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = inet_addr("127.0.0.1");
addr.sin_port = htons(8080); 

if (bind(sockfd, (struct sockaddr *) &addr, sizeof(addr)) < 0) {
    perror("Failed to bind socket");
    return 1;
};
```

## Transforming the socket into a passive socket
Next, we will use [`Listen(2)`](https://man7.org/linux/man-pages/man2/listen.2.html) to transform the socket into a passive socket, used to accept incoming connections request using `accept(2)`. We will use `128` as the backlog size for now. On success, 0 is returned. We need to handle -1 on error and handle `errorno`.

```c
#include <sys/socket.h>
int listen(int sockfd, int backlog);

// in main
int listenBacklog = 1;
if (listen(sockfd, listenBacklog) < 0) {
    perror("Failed to listen on port 8080");
    return 1;
}
```

## Accepting incoming connections & read data
We can use `accept()` to take the first pending connection from the queue of the server socket and create a new connected socket. `accept` will return the file descriptor of the new socket.

```c
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *_Nullable restrict addr, socklen_t *_Nullable restrict addrlen);
```

Since everything in linux is a file, we can use the `read()` to retrieve the data into a buffer, and print out the result.

```c
char buffer[BUFFER_SIZE];
ssize_t data_len = read(clientfd, buffer, BUFFER_SIZE);
log_with_time("received data (%d):[%sb]", data_len, buffer);
```

We can send a small packet with `nc 127.0.0.1:8080 < data`, and the complete program output is the following
```
$ nc 127.0.0.1:8080 < data

[2025-01-30 15:38:48] openned socket with fd:3
[2025-01-30 15:38:48] waiting for connection at port:8080
[2025-01-30 15:38:49] received connection, new socket:4, client address:127.0.0.1
[2025-01-30 15:38:49] received data (12):[Hi! I am Bobb]
[2025-01-30 15:38:49] closed socket with fd:3
```

