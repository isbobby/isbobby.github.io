---
title: TCP Part 1 - Writing a server in C
parent: Computer Network
nav_order: 2
---
# Writing a TCP server in C
The basic TCP server program should do the following, code is pushed [here](https://github.com/isbobby/sockets-programming/blob/main/c/server.c).

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

Since everything in linux is a file, we can use the `read()` to retrieve the data into a buffer, and print out the result. After reading, we need to close the client socket with `close(clientfd)`

```c
char buffer[BUFFER_SIZE];
ssize_t data_len = read(clientfd, buffer, BUFFER_SIZE);
log_with_time("received data (%d):[%sb]", data_len, buffer);
close(clientfd);
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

## Long running process
The server program exits every time it serves a request. We can let it continuously wait for incoming connection with a loop.

The final server program will look like the following
```c
#include <sys/socket.h>
#include <unistd.h>
#include <stdio.h>
#include <netinet/in.h>
#include <arpa/inet.h>


#include "log.h"

#define PORT 8080
#define BUFFER_SIZE 1024

// main driver function
// 1. create a socket
// 2. we can configure socket, but do nothing here first
// 3. close socket 
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
    if (bind(sockfd, (struct sockaddr *) &addr, sizeof(addr)) < 0) {
        perror("Failed to bind socket");
        return 1;
    };

    int listenBacklog = 1;
    if (listen(sockfd, listenBacklog) < 0) {
        perror("Failed to listen on port");
        return 1;
    }

    socklen_t client_addr_size;
    struct sockaddr_in client_addr;
    client_addr_size = sizeof(client_addr);
    log_with_time("waiting for connection at port:%d", PORT);
    for (;;) {
        int clientfd = accept(sockfd, (struct sockaddr *) &client_addr, &client_addr_size);
        if (clientfd == -1) {
            perror("Failed to accept client connection");
            return 1;
        }

        char client_addr_str[INET_ADDRSTRLEN];
        inet_ntop(AF_INET, &client_addr.sin_addr, client_addr_str, INET_ADDRSTRLEN);
        log_with_time("received connection, new socket:%d, client address:%s", clientfd, client_addr_str);
        
        char buffer[BUFFER_SIZE];
        ssize_t data_len = read(clientfd, buffer, BUFFER_SIZE);
        log_with_time("received data (%d):[%sb], sending response to client", data_len, buffer);

        char write_back[1024];
        sprintf(write_back, "received data (%zdb)", data_len);
        if (write(clientfd, write_back, strlen(write_back)) < 0) {
            log_with_time("failed to respond to client");
        }

        close(clientfd);
    }
    
    log_with_time("closed socket with fd:%d", sockfd);
    close(sockfd);

    return 0;
}
```

The `log.h` file defines the helper function to log with timestamp.

```c
#include <time.h>
#include <stdarg.h>
#include <string.h>
#include <stdio.h>

void log_with_time(char *format, ...) {
    int time_stamp_size = 23;
    int new_line_size = 2;

    char time_buff[time_stamp_size];
    time_t rawtime = time(0);
    strftime(time_buff, time_stamp_size, "[%Y-%m-%d %H:%M:%S] ", localtime(&rawtime));

    char log_buff[time_stamp_size + strlen(format) + new_line_size];
    strcpy(log_buff, time_buff);
    strcat(log_buff, format);
    strcat(log_buff, "\n");

    va_list args;
    va_start(args, format);
    vprintf(log_buff, args);
    va_end(args);
}
```
