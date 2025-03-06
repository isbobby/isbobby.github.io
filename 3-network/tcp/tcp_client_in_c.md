---
title: TCP Part 2 - Writing a client in C
parent: TCP
nav_order: 3
---
# Part 2 - TCP Client
Once we have a process listening at a TCP port, we can send a message to the server with a client process.

The client process will need to
1. create a socket with the same `socket()` and get the `fd`, same as the server
2. initiate and establish connection with `connect()`
3. write data to the socket after `connect()` with `write()`
4. retrieve the server's response with `read()`

## Connecting to a server socket
We need to connect the client socket to the destination using [`connect(2)`](https://man7.org/linux/man-pages/man2/connect.2.html).
```c
#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

We have encountered the [`sockaddr`](https://man7.org/linux/man-pages/man3/sockaddr.3type.html) type before, we will use the `sockaddr_in` type for internet socket.

```c
struct sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = inet_addr("127.0.0.1");
addr.sin_port = htons(PORT); 

if (connect(sockfd, (struct sockaddr *) &addr, sizeof(addr)) < 0) {
    perror("Failed to connnect to server socket");
    return 1;
};
```
## Socket I/O
After connecting to the server, we can do `read()` and `write()` on the socket. For now, we will be using a `1024` (1kb) buffer.

```c
#define BUFFER_SIZE 1024

// hard coded data for now
char * data = "(hard coded) Hi!";
if (write(sockfd, data, strlen(data)) < 0) {
    perror("failed to write data");
    return 1;
}
log_with_time("wrote data (%db) to server", strlen(data));

char * recv_data[BUFFER_SIZE];
if (read(sockfd, recv_data, BUFFER_SIZE) < 0) {
    perror("failed to read server response");
}
log_with_time("server response:%s", recv_data);
```

The final client program will be (the `log.h` file is in the previous document)
```c
#define PORT 8080

#include <unistd.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>

#include "log.h"

#define BUFFER_SIZE 1024

int main() {
    int sockfd;
    sockfd = socket(AF_INET, SOCK_STREAM, 6);
    if (sockfd == -1) {
        perror("Error opening socket");
        return 1;
    }
    log_with_time("openned socket with fd:%d", sockfd);

    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    addr.sin_port = htons(PORT); 
    if (connect(sockfd, (struct sockaddr *) &addr, sizeof(addr)) < 0) {
        perror("Failed to connnect to server socket");
        return 1;
    };

    // hard coded data for now
    char * data = "(hard coded) Hi!";
    if (write(sockfd, data, strlen(data)) < 0) {
        perror("failed to write data");
        return 1;
    }
    log_with_time("wrote data (%db) to server", strlen(data));
    
    char * recv_data[BUFFER_SIZE];
    if (read(sockfd, recv_data, BUFFER_SIZE) < 0) {
        perror("failed to read server response");
    }
    log_with_time("server response:%s", recv_data);
    
    return 0;
}
```
