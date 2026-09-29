# Network Programming with C UNIX

## 1. What are sockets?
### 1.1. Socket definition
Sockets are an API at the **transport layer** in the **TCP/IP** network stack.

Applications send and receive data through the socket.

On the same server, sockets having different usage and services are identified via their **port number**.

Instead of raw IPv4, the user can set up the socket to have its own human-readable hostname.

### 1.2. UNIX sockets
UNIX sockets generally uses ```send``` for sending data and ```recv``` for reading.

UNIX sockets can be **non-blocking** or **blocking** running multiple logics, and can set up to have **multithreading** and **multiprocessing** for server efficiency and reliability.

### 1.3. Socket protocols

```TCP``` is the end to end protocol providing a reliable byte-stream (data transfer) service, thus it's used for data transfer systems.

```UDP``` instead provide a best effort datagram service.

Socket addresses are often in the ```ip:port``` form for IPv4.

TCP provides connections between clients and servers, provides reliability via:

1. **Acknowledgement**: The server notifies itself on receiving data.
2. **Error control**: Easy to track and setup error scenarios.
3. **Flow control**: Controls the flow, preventing deadlock processes.

### 1.4. General TCP working flow

                    SERVER                         CLIENT
                      │                              │
                  socket()                       socket()
                      │                              │
                   bind()                           │
                      │                              │
                  listen()                          │
                      │                              │
                  accept() ◄──── TCP connection ─── connect()
                      │                              │
                      │                              │
                 recv()/read() ◄──────────────── send()/write()
                      │                              │
                 send()/write() ──────────────────► recv()/read()
                      │                              │
                      │                              │
                  close() ◄──── connection ─────── close()
                      │                              │

### 1.5. General UDP working flow
                         UDP SERVER                    UDP CLIENT
                             │                             │
                         socket()                      socket()
                             │                             │
                          bind()                           │
                             │                             │
                             │                         sendto()
                             │◄────────────────────────────│
                         recvfrom()                         │
                             │                             │
                         sendto() ────────────────────────►│
                             │                         recvfrom()
                             │                             │
                             │                         sendto()
                             │◄────────────────────────────│
                         recvfrom()                         │
                             │                             │
                          close()                       close()


## 2. Basics
### 2.1. Headers
Commonly used headers:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>

#include <unistd.h>
#include <fcntl.h>

#include <sys/types.h>
#include <sys/socket.h>
#include <sys/select.h>

#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>

#include <poll.h>
#include <signal.h>
```

### 2.2. Socket address structure
### 2.2.1. Initialization

There are three main address structures for a socket.
#### 1. `struct sockaddr`: 
Is the general structure, can be used to represent both IPv4 and IPv6 addresses. Must be casted into the other two types to be able to pass parameters into the socket, or the IPv4 and IPv6 types are casted back to `sockaddr`
```c
struct sockaddr {
    sa_family_t sa_family;
    char        sa_data[14];
};
```

Example: Casting to `sockaddr_in` or `sockaddr_in6` for domain resolving.
```c
struct sockaddr *addr;
struct sockaddr_in *ipv4 = (struct sockaddr_in *)addr;
```

Example: Casting `sockaddr_in` back to `sockaddr` for binding.
```c
struct sockaddr_in addr;
// Add parameters to addr
bind(server_fd, (struct sockaddr *)&addr, sizeof(addr));
```

#### 2. `struct sockaddr_in`: 
Used for IPv4 processing.
```c
struct sockaddr_in {
    uint_8 sin_len;                 // Structure length (16 bytes)
    sa_family_t sin_family;         // IPv4 ip family
    in_port_t sin_port;             // Port number (0 to 65535)
    struct in_addr sin_addr;        // 32 bits of IPv4 address
    char sin_zero[8];               // Unused field
}
```
- ```sin_family``` : Used to determine the type of IP. ```AF_INET``` for IPv4 and ```AF_INET6``` for IPv6.

- ```sin_port``` : The bytes of the port number, from 0 to 65535. Convert an **integer** representing port number to network bytes using ```htons(port_num)```.

- ```sin_addr``` : Contains the IP of the socket, modify the ```s_addr``` of this. Use ```INADDR_ANY``` to allow for any connections from any IP or use ```inet_addr(ip_string)``` to determine the IP of the socket for the client side.

#### 3. `struct sockaddr_in6`: Used for IPv6 processing.


Usually, at least for IPv4, we use the data type ```sockaddr_in``` from the library ```netinet/in.h``` to provide an address to an IPv4 socket.

Example of casting sockaddr to IPv4 type, setting up the socket for server and client side:
```c
// Casting to IPv4
struct sockaddr *addr;
struct sockaddr_in *ipv4 = (struct sockaddr_in *)addr;

// Server side socket
struct sockaddr_in addr;
addr.sin_family = AF_INET;                                      // IPv4
addr.sin_addr.s_addr = INADDR_ANY                               // Accept connection from any address
addr.sin_port = htons(8080)                                     // Port is 8080

// Client side socket
struct sockaddr_in addr;
addr.sin_family = AF_INET;                                      // IPv4
addr.sin_addr.s_addr = inet_addr("192.168.32.16")               // Client address
addr.sin_port = htons(8080)                                     // Port is 8080
```

### 2.3. Domain resolving
#### 2.3.1. Data structures
Address information are saved under the ```addrinfo``` struct.
```c
#include <netdb.h>
struct addrinfo {
    int              ai_flags;              
    int              ai_family;                 // Address family (AF_INET or AF_INET6)
    int              ai_socktype;               // Socket type (SOCK_STREAM or SOCK_DGRAM)
    int              ai_protocol;               // The protocol (IPPROTO_TCP or IPPROTO_UDP)
    socklen_t         ai_addrlen;               
    struct sockaddr  *ai_addr;                  // Contains the actual address (sockaddr_in or sockaddr_in6)
    char             *ai_canonname;             // Domain name
    struct addrinfo  *ai_next;                  // Pointer to the next element in the linked list
};
```
- `ai_family` can take on either `AF_INET` for IPv4 or `AF_INET6` for IPv6.
```c
struct addrinfo addr;
addr.ai_family = AF_INET;
addr.ai_family = AF_INET6;
```
- `ai_addr` contains the pointer to the address structure of the addrinfo, which can be casted into `sockaddr_in` or `sockaddr_in6`.

Example of casting a `sockaddr` into its IPv4 type.
```c
struct sockaddr *p = addrinfo.ai_addr;
struct sockaddr_in *out;
out = (struct sockaddr_in *)p;
char ip[INET_ADDRSTRLEN];                               // Built-in variable for IPv4 string length
inet_ntop(AF_INET, &out->sin_addr, ip, sizeof(ip));     // Extract IP address
int port = ntohs(out->sin_port);                        // Extract port number
```

- `ai_next` points to the next element if the resolved linked list of addresses.

#### 2.3.2. Resolving

To resolve the address of a domain, we use the function `getaddrinfo()`.
```c
int getaddrinfo(
    const char* domain,                 // Host name to resolve (google.com)
    const char* servname,               // Service ("http") or port ("80")
    const struct addrinfo* hints,       // Pointer to the hints containing the search constraints
    struct addrinfo** res               // Pointer to the first element of the linked list
);                 
```
Returns `0` if success, `!= 0` if failed.

Example of a function looking for both IPv4 and IPv6 addresses from a domain.
```c
void resolve(char domain[]) {
    struct addrinfo hints = {0};        // Hints parameter
    struct addrinfo *res;               // Result linked list

    hints.ai_family = AF_UNSPEC         // Accept both IPv4 and IPv6
    hints.ai_socktype = SOCK_STREAM     // Set socket type to TCP

    int status = getaddrinfo(domain, "http", &hints, &res);
    if (status != 0 || res == NULL) {
        return;
    }

    struct addrinfo *cur = res;         // Pointer to current node (1st node)
    while (cur != NULL) {
        // Prints IPv4 value if the address type is IPv4
        if (cur->ai_family == AF_INET) {
            printf("IPv4\n");
            struct sockaddr_in *addr = (struct sockaddr_in *)cur->ai_addr;
            char ip[INET_ADDRSTRLEN];
            inet_ntop(AF_INET, &addr->sin_addr, ip, sizeof(ip));
            printf("Address: %s\n", ip);
            printf("Port: %d\n", ntohs(addr->sin_port));
        // Print IPv6 value if the address type is IPv6
        } else if (cur->ai_family == AF_INET6) {
            printf("IPv6\n");
            struct sockaddr_in6 *addr = (struct sockaddr_in6 *)cur->ai_addr;
            char ip[INET6_ADDRSTRLEN];
            inet_ntop(AF_INET6, &addr->sin6_addr, ip, sizeof(ip));

            printf("Address: %s\n", ip);
            printf("Port: %d\n", ntohs(addr->sin6_port));
        }

        cur = cur->ai_next;
    }
}
```

### 2.4. TCP Network Programming
#### 2.4.1. Common functions
#### 1. `socket()`:
UNIX sockets are initialized as file descriptors (integers in UNIX), representing the created socket.

The function `socket()` is used to perform such a task, from the `sys/socket.h` library.

```c
int sock_fd = socket(AF_INET, SOCK_STREAM, 0);          // Creating a TCP IPv4 socket
int sock_fd6 = socket(AF_INET6, SOCK_STREAM, 0);        // Creating a TCP IPv6 socket
int udpsock_fd = socket(AF_INET, SOCK_DGRAM, 0);        // Creating an UDP IPv4 socket
```

`socket()` takes in three parameters:
- `int domain`: The domain of the socket connection. Can be `AF_INET` or `AF_INET6` for either IPv4 or IPv6.
- `int type`: The type of the socket. Can be `SOCK_STREAM` for TCP or `SOCK_DGRAM` for UDP.
- `int protocol`: The protocol of the socket. Usually set to `0` for the OS to choose the domain and type automatically.

However, before setting up the socket for an actual connection, we need to set up the addresses for the socket, this is where the socket address structures come into handy.

#### 2. `sockaddr_in`:
As stated above in the [socket structure](#221-initialization) part, the `sockaddr_in` is the structure containing all of the variables for the socket.
```c
// Server side socket
struct sockaddr_in addr;
addr.sin_family = AF_INET;                                      // IPv4
addr.sin_addr.s_addr = INADDR_ANY                               // Accept connection from any address
addr.sin_port = htons(8080)                                     // Port is 8080

// Client side socket
struct sockaddr_in addr;
addr.sin_family = AF_INET;                                      // IPv4
addr.sin_addr.s_addr = inet_addr("192.168.32.16")               // Client address
addr.sin_port = htons(8080)                                     // Port is 8080
```

#### 3. `bind()`:
After setting up the address structure, the `bind()` function is called to bind that address to the initialized socket.
```c
int status = bind(sock_fd, (struct sockaddr *)&addr, sizeof(addr));
```
This function takes in three parameters:
- `int sock_fd`: The socket to bind to, initialized via `socket()`.
- `const struct sockaddr *addr`: The socket address structure, casted to `sockaddr` from `sockaddr_in`.
- `socklen_t addrlen`: The size of the address, use `sizeof()`.

`bind()` returns `0` if success, `-1` if fail.

#### 4. `listen()` and `accept()` (server-side only):
`listen()` is used to setup a server-side socket into listening for incoming connections from clients.
```c
int status = listen(sock_fd, 10);
```
This function takes in two parameters:
- `int sock_fd`: The socket to bind to, initialized via `socket()`.
- `int backlog`: Number of queued connections allowed at once.

While, `accept()` is used to accept incoming connections from other clients.
```c
struct sockaddr_in client_addr;                                                       // Set up address structure of client
socklen_t client_len = sizeof(client_addr);                                           // Size of client_addr structure
int client_fd = accept(server_fd, (struct sockaddr *)&client_addr, &client_len);      // Accept new clients from here
```
This function takes in three parameters:
- `int server_fd`: The server's socket.
- `const struct sockaddr *client_addr`: The pointer to the socket address structure of the client, casted to `sockaddr` from `sockaddr_in`.
- `socklen_t *client_len`: The pointer to the size of the address, use `sizeof()` before hand (do not call directly in function).

Returns a value greater than `0` if success, `<0` if fail.
This function goes **inside** the `while` loop of the server.

#### 5. `connect()` (client-side only):
```c
struct sockaddr_in server_addr = {0};                          // Initialize address structure of server
server_addr.sin_family = AF_INET;                              // Set up protocol, port and IP.
server_addr.sin_port = htons(8080);
inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr);
int status = connect(client_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));  // Perform connection to server
```

This function takes in three parameters:
- `int client_fd`: The client's socket.
- `const struct sockaddr *server_addr`: The pointer to the socket address structure of the server, casted to `sockaddr` from `sockaddr_in`.
- `socklen_t addrlen`: The pointer to the size of the address, use `sizeof()` inside the function call.

#### 6. `send()`:
`send()` is used to send data through a connected socket.
```c
ssize_t send(int sock_fd, const void buf*, size_t len, int flags);
```
This function takes in four variables:
- `int sock_fd`: The socket you want to send data with. 
- `const void *buf`: The pointer to the first byte you want to send, can be any type of data.
- `size_t len`: The number of bytes to send through. Often set to buffer size if exist, can use `sizeof()` if want to send all data at once.
- `int flags`: Flags, usually set to `0`.

This function returns the number of bytes sent via the function call, returns `0` if connection closed or `-1` if there was an error.

In real applications, a buffer is used as there's no guarantee the entire data is sent in complete due to system or network limits. Thus, a `send_all` function can be used to send all data at once.
```c
void send_all(int client_fd, char response[]) {
    size_t total = strlen(response);                                          // Total number of bytes of the data to send
    size_t total_sent = 0;                                                    // Total bytes already sent

    while (total_sent < total) {                                              
        size_t remaining = total - total_sent;                                // Reduce the remaining number of bytes left to send
        size_t buffer = remaining < BUFFER_SIZE ? remaining : BUFFER_SIZE;    // Data buffer to be sent in this loop (send full buffer size if remaining > buffer size, directly if not)

        ssize_t sent = send(client_fd, response + total_sent, buffer, 0);     // Send the data buffer (response + total_sent means advance the data by number of bytes send then send)

        if (sent <= 0) {
            if (sent < 0) perror("send");
            break;
        }
        total_sent += sent;
    }
}
```

#### 7. `recv()`:
Conversely, `recv()` is the function used for the socket to fetch data sent to it.
```c
ssize_t recv(int sock_fd, void buf*, size_t len, int flags);
```
This function takes in four variables:
- `int sock_fd`: The socket you want to send data with. 
- `const void *buf`: The pointer to the buffer you want to receive data and put them into.
- `size_t len`: The number of bytes to receive. Often set to buffer size if exist, can use `sizeof()` if want to send all data at once.
- `int flags`: Flags, usually set to `0`.

When receiving a string, and for the case is that data is received on one `recv()` call, it is usually best practice to receive data in size of `buffer size - 1`, leaving the last byte for the null terminator just in case.
```c
char buffer[BUFF_SIZE];
ssize_t n = recv(sock_fd, buffer, BUFF_SIZE - 1, 0);
if (n > 0) {
  buffer[n] = '\0';       // Set index n + 1 to be null terminator
}
```

You can also use a loop to keep calling `recv()` until all data is received.
```c
while ((n = recv(client_fd, buffer, sizeof(buffer) - 1, 0)) > 0) {
  buffer[n] = '\0';
  printf("%s", buffer);
}
```

### 2.5. UDP Network Programming
We initiate the socket via `socket()` and use `struct sockaddr_in` with `bind()` like with TCP, but with important differences.



### 2.6. Utility functions

You can retrieve the IPv4 string format from the network bytes using ```inet_ntoa```.
```c
char *ip = inet_ntoa(sin_addr);                                 // Returns the IP string like "127.0.0.1"
```

Additionally, ```inet_ntop``` can also be used, returning NULL if error.
```c
struct sockaddr_in *addr;                                  // Initialize a pointer to the socket address structure
char ip[];
inet_ntop(AF_INET, &addr->sin_addr, ip, sizeof(ip));       // Writes the IP string into the string variable
```

The port number can be extracted via ```ntohs()```.
```c
int port = ntohs(addr.sin_port);
```
