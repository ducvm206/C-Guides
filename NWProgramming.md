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
##### 1. `socket()`:
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

##### 2. `sockaddr_in`:
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

##### 3. `bind()`:
After setting up the address structure, the `bind()` function is called to bind that address to the initialized socket.
```c
int status = bind(sock_fd, (struct sockaddr *)&addr, sizeof(addr));
```
This function takes in three parameters:
- `int sock_fd`: The socket to bind to, initialized via `socket()`.
- `const struct sockaddr *addr`: The socket address structure, casted to `sockaddr` from `sockaddr_in`.
- `socklen_t addrlen`: The size of the address, use `sizeof()`.

`bind()` returns `0` if success, `-1` if fail.

##### 4. `listen()` and `accept()` (server-side only):
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

##### 5. `connect()` (client-side only):
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

##### 6. `send()`:
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

##### 7. `recv()`:
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

#### 2.4.2. Non-blocking network
A **blocking socket** makes the program wait at a function such as `accept()` or `recv()` until the operation can proceed. A **non-blocking socket** instead returns immediately when no connection or data is available.

```text
BLOCKING                              NON-BLOCKING
recv()                                recv()
  │                                     │
  ├── No data: WAIT                     ├── No data: return -1 (EAGAIN)
  │       ...                           │
  └── Data arrives → return            └── Do other work → retry later
```

Non-blocking does **not** mean data is transferred faster. It means one socket operation does not force the entire thread to wait. A program can therefore work on other connections while a client is idle.

##### 1. `ioctl()` — changing a socket's behavior
From `<sys/ioctl.h>`, `ioctl()` controls device/file-descriptor behavior. For sockets, `FIONBIO` enables or disables non-blocking mode.

```c
#include <sys/ioctl.h>
int ioctl(int fd, unsigned long request, ...); // Request-specific third argument
```

- `fd`: Socket file descriptor returned by `socket()` or `accept()`.
- `request`: `FIONBIO` means **set non-blocking mode** according to an integer value.
- `...`: For `FIONBIO`, pass a pointer to an `int`: nonzero enables non-blocking; zero restores blocking.
- Returns `0` on success and `-1` on error (`errno` indicates why).

```c
int nonblocking = 1;
if (ioctl(client_fd, FIONBIO, &nonblocking) == -1) {
    perror("ioctl FIONBIO");
}
// To return to blocking mode:
nonblocking = 0;
ioctl(client_fd, FIONBIO, &nonblocking);
```

Unlike `fcntl(F_GETFL/F_SETFL)`, `ioctl(FIONBIO)` directly changes the socket's blocking behavior using an integer pointer. Apply it to each accepted client socket when needed; do not assume the listening socket's mode is inherited on every platform.

##### 2. `errno`, `EAGAIN`, and `EWOULDBLOCK`
From `<errno.h>`, `errno` is an error code set when a system call fails. With a non-blocking socket, `recv()` may return `-1` with `errno == EAGAIN` or `errno == EWOULDBLOCK`, meaning **no data is available right now**, not that the connection has broken.

```c
#include <errno.h>
char buffer[1024];
ssize_t n = recv(client_fd, buffer, sizeof(buffer) - 1, 0);
if (n > 0) {
    buffer[n] = '\0';                              // Data received
    printf("Received: %s\n", buffer);
} else if (n == 0) {
    close(client_fd);                               // Peer closed connection
} else if (errno == EAGAIN || errno == EWOULDBLOCK) {
    // No data yet: work on another socket, or wait for readiness
} else if (errno == EINTR) {
    // Interrupted by a signal: retry when appropriate
} else {
    perror("recv");
}
```

`send()` can also return `-1` with `EAGAIN` / `EWOULDBLOCK`, or send **only part** of a buffer. Save the unsent bytes and retry when the socket is writable. A non-blocking `connect()` may return `-1` with `EINPROGRESS`; the connection is still in progress and must be checked later (for example with `select()` and `getsockopt(SO_ERROR)`).

**Important:** Constantly retrying `recv()` in a tight loop wastes CPU (**busy waiting**). Use `select()` (section 2.4.5) to sleep until a socket is ready.

##### 3. Real usage: a chat client that checks messages without freezing
Imagine a chat client that must update its screen even when the server has not sent anything. `ioctl(FIONBIO)` lets `recv()` check for incoming text and return immediately when the inbox is empty.

```c
// Place after connect(client_fd, ...) succeeds.
#include <sys/ioctl.h>
#include <errno.h>
#include <unistd.h>

int on = 1;
if (ioctl(client_fd, FIONBIO, &on) == -1) { perror("ioctl"); return 1; }
for (int tick = 0; tick < 5; tick++) {
    char msg[256];
    ssize_t n = recv(client_fd, msg, sizeof(msg) - 1, 0);
    if (n > 0) { msg[n] = '\0'; printf("Chat message: %s\n", msg); }
    else if (n == 0) { puts("Server disconnected"); break; }
    else if (errno != EAGAIN && errno != EWOULDBLOCK && errno != EINTR) {
        perror("recv"); break;
    }
    printf("UI tick %d: still responsive\n", tick);
    sleep(1); // Demo only: real applications use select()/poll() for readiness
}
```

Example timeline (server sends `Hello` during tick 2):

```text
Time       recv() result             Other work
0 s        -1, EAGAIN                UI tick 0
1 s        -1, EAGAIN                UI tick 1
2 s         5 bytes: "Hello"         UI tick 2
3 s        -1, EAGAIN                UI tick 3
```

The socket remains connected when `EAGAIN` occurs. This example assumes an already-connected TCP socket and treats each read as a displayable chunk, not necessarily a complete chat message.


#### 2.4.3. Multi-processing network
A **multi-process server** creates a separate **process** to handle a client. Each child process has its own address space, so a crash in one child is less likely to affect another client.

```text
                     SERVER (parent process)
                               │
                         accept() client A
                               │
                             fork()
                          ┌────┴────┐
                       Parent     Child A ── recv()/send() A
                          │
                     accept() client B
                          │
                        fork()
                     ┌────┴────┐
                  Parent     Child B ── recv()/send() B
```

Unlike non-blocking sockets, each child can use ordinary **blocking** `recv()` because other clients are handled by other processes. Processes have more memory and creation overhead than threads; ordinary variables are **not shared** between them.

##### 1. `fork()` — create a child process
From `<unistd.h>`:

```c
#include <unistd.h>
pid_t fork(void);
```

- Called once, but execution continues in **two processes**.
- Returns `> 0` (child PID) in the **parent**, `0` in the **child**, or `-1` if creation failed.
- The child inherits copies of the parent's file descriptors, referring to the same underlying open sockets.

##### 2. `pid_t` — process ID type
From `<sys/types.h>`, `pid_t` is an integer type used to store process IDs. Its exact underlying integer type is system-dependent.

```c
pid_t pid = fork();
if (pid == -1) {
    perror("fork");
} else if (pid == 0) {
    // Child process
} else {
    // Parent process; pid identifies the child
}
```

##### 3. `_exit()` and `waitpid()` — ending and collecting children
A child should exit when it finishes its client. `_exit()` from `<unistd.h>` terminates immediately, without flushing inherited C stdio buffers.

```c
void _exit(int status);               // 0 generally indicates success
```

The parent must also **reap** finished children, or they remain as zombie processes. From `<sys/wait.h>`:

```c
#include <sys/wait.h>
pid_t waitpid(pid_t pid, int *status, int options);
```

- `pid`: `-1` means any child; positive value means one specific child.
- `status`: Pointer where child termination information is stored; `NULL` if not needed.
- `options`: `WNOHANG` makes the call return immediately if no child has exited.
- Returns child PID when reaped, `0` if `WNOHANG` found no finished child, `-1` on error.

##### 4. Example: fork-per-client TCP server
This example assumes `server_fd` has already been created, bound and put into listening mode using section 2.4.1.

```c
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <errno.h>
#include <stdio.h>
#include <sys/socket.h>

while (1) {
    // Collect children that have already exited (non-blocking)
    while (waitpid(-1, NULL, WNOHANG) > 0) {}

    int client_fd = accept(server_fd, NULL, NULL);
    if (client_fd == -1) {
        if (errno == EINTR) continue;
        perror("accept");
        continue;
    }

    pid_t pid = fork();
    if (pid == -1) {
        perror("fork");
        close(client_fd);
        continue;
    }
    if (pid == 0) {                              // CHILD
        close(server_fd);                        // Child doesn't accept clients
        char buffer[1024];
        ssize_t n = recv(client_fd, buffer, sizeof(buffer), 0);
        if (n > 0) send(client_fd, buffer, (size_t)n, 0); // Simple echo
        close(client_fd);
        _exit(0);
    }
    close(client_fd);                            // PARENT: child owns its copy
}
```

The echo example performs one `recv()` and one `send()` for clarity; real TCP applications loop for complete messages and partial sends. For a production server, arrange child reaping even while `accept()` is blocked (for example with a `SIGCHLD` handler or event loop), and handle shutdown cleanly.

##### 5. Real usage: several customers requesting files simultaneously
A file-download server can `fork()` for each incoming customer. Each child performs a potentially slow `send()` independently, while the parent immediately returns to `accept()`.

```text
Parent PID 4000: accept() ── client A ── fork() ── Child PID 4001 ── send(file A)
       │
       └─────────────── client B ── fork() ── Child PID 4002 ── send(file B)
       │
       └─────────────── client C ── fork() ── Child PID 4003 ── send(file C)
```

Inside the existing fork-per-client example, replace the child's echo operation with a fixed demonstration response:

```c
if (pid == 0) {                  // Child handles only its own client
    close(server_fd);            // Child does not accept new clients
    const char *reply = "Your download is being prepared\n";
    send(client_fd, reply, strlen(reply), 0); // Demo: production checks partial sends
    close(client_fd);
    _exit(0);
}
close(client_fd);                // Parent keeps listening
```

**Observable result:** A slow client A does not prevent the parent from accepting B. The child processes have separate memory; a counter incremented in child A will not update child B's counter. The snippet belongs inside the existing server's `accept()` loop after `pid = fork()`; handle `fork() == -1` separately and reap children as shown above.


#### 2.4.4. Multi-threading network
A **multi-threaded server** creates multiple **threads inside one process**. Threads share global variables, heap memory and file descriptors, but each has its own stack and execution flow.

```text
                    ONE SERVER PROCESS
                ┌─────────────────────────┐
                │ Main thread: accept()   │
                │     ├── Worker A → A    │
                │     └── Worker B → B    │
                │ Shared heap / globals   │
                └─────────────────────────┘
```

Unlike `fork()`, threads do not create separate address spaces, so sharing a counter or cache is easier. However, simultaneous access to shared data can cause **race conditions**, and a process crash affects every thread. A blocking `recv()` in worker A does not block worker B.

Compile pthread programs on UNIX with `-pthread` (e.g. `gcc server.c -o server -pthread`).

##### 1. `pthread_t` and `pthread_create()`
From `<pthread.h>`, `pthread_t` is an opaque type identifying a thread. Do not assume it is an `int`.

```c
#include <pthread.h>
int pthread_create(pthread_t *thread,
                   const pthread_attr_t *attr,
                   void *(*start_routine)(void *),
                   void *arg);
```

- `thread`: Output location for the created thread's ID.
- `attr`: Thread configuration; `NULL` uses defaults. `pthread_attr_t` is an opaque configuration type.
- `start_routine`: Function run by the new thread. It must accept `void *` and return `void *`.
- `arg`: Pointer passed to that function (often a pointer to a client descriptor or a struct).
- Returns `0` on success, otherwise an **error number** (not necessarily via `errno`).

##### 2. `pthread_detach()` and `pthread_join()`

```c
int pthread_detach(pthread_t thread);
int pthread_join(pthread_t thread, void **retval);
```

- `pthread_detach(thread)`: Releases thread resources automatically when it finishes; a detached thread cannot be joined.
- `pthread_join(thread, &result)`: Waits until a joinable thread finishes and optionally obtains its return pointer. Use `NULL` when the result is not needed.
- Both return `0` on success or an error number otherwise.

##### 3. `pthread_mutex_t` — protecting shared data
A **mutex** is a lock: only one thread can hold it at a time. Other threads trying to lock it must wait.

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

- `pthread_mutex_t`: Opaque mutex object; initialize statically as above, or dynamically using `pthread_mutex_init()`.
- `pthread_mutex_lock()`: Acquire the lock before modifying shared data.
- `pthread_mutex_unlock()`: Release the lock afterward.
- `pthread_mutex_destroy()`: Clean up an unlocked mutex when it is no longer used.
- These methods return `0` on success or an error number.

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
int total_clients = 0;                       // Shared between threads

// Inside worker thread:
pthread_mutex_lock(&lock);
total_clients++;                             // Protected shared update
pthread_mutex_unlock(&lock);
```

##### 4. Example: thread-per-client TCP server
This example assumes `server_fd` is already listening. `malloc()` gives each worker a **different memory location** for its client descriptor; passing `&client_fd` (a loop-local variable reused by the main thread) would be unsafe.

```c
#include <pthread.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/socket.h>
#include <unistd.h>

void *handle_client(void *arg) {
    int client_fd = *(int *)arg;
    free(arg);                                 // Worker owns this allocation

    char buffer[1024];
    ssize_t n = recv(client_fd, buffer, sizeof(buffer), 0);
    if (n > 0) send(client_fd, buffer, (size_t)n, 0);
    close(client_fd);
    return NULL;
}

while (1) {
    int client_fd = accept(server_fd, NULL, NULL);
    if (client_fd == -1) { perror("accept"); continue; }

    int *arg = malloc(sizeof(*arg));
    if (arg == NULL) { close(client_fd); continue; }
    *arg = client_fd;

    pthread_t thread;
    int err = pthread_create(&thread, NULL, handle_client, arg);
    if (err != 0) {
        fprintf(stderr, "pthread_create error: %d\n", err);
        free(arg);
        close(client_fd);
        continue;
    }
    pthread_detach(thread);                    // No join required
}
```

As above, this is a one-read/one-send teaching example, not a full message framing implementation. Thread-per-client designs should limit concurrent threads under heavy load (for example using a worker pool).

##### 5. Real usage: multiple players in a multiplayer game lobby
A thread-per-client server gives each player a worker thread. All threads can access the same lobby state, so a mutex protects shared player counts.

```text
Server process (one shared memory space)
 ├── Thread A: recv(player A) ─┐
 ├── Thread B: recv(player B) ─┼── shared player_count
 └── Thread C: recv(player C) ─┘          ▲
                                    mutex lock
```

The following worker illustrates the critical section (pass a heap-allocated `int *` containing the accepted socket descriptor, as in the full server example):

```c
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/socket.h>

pthread_mutex_t lobby_lock = PTHREAD_MUTEX_INITIALIZER;
int player_count = 0;

void *player_worker(void *arg) {
    int fd = *(int *)arg;
    free(arg);
    pthread_mutex_lock(&lobby_lock);
    ++player_count;               // Only one thread updates count at a time
    pthread_mutex_unlock(&lobby_lock);

    char buf[256];
    while (recv(fd, buf, sizeof(buf), 0) > 0) {
        // Handle this player's messages (simplified example).
    }
    pthread_mutex_lock(&lobby_lock);
    --player_count;
    pthread_mutex_unlock(&lobby_lock);
    close(fd);
    return NULL;
}
```

**Observable result:** If A waits 10 seconds before sending data, B's thread can still receive immediately. Compile a full server using this worker with `gcc server.c -o server -pthread`. Never hold the lobby mutex while waiting inside `recv()`.


#### 2.4.5. `select()`
`select()` is an **I/O multiplexing** function: **one thread watches multiple socket descriptors** and sleeps until at least one is ready for reading, writing, or has an exceptional condition.

```text
Client A ── fd 4 ──┐
Client B ── fd 5 ──┼── select() ── ready: fd 5 ── recv(fd 5)
Client C ── fd 6 ──┘       │
                         wait until any socket is ready
```

Unlike multiprocessing or multithreading, `select()` does not create extra workers. Unlike busy-loop non-blocking I/O, it can wait efficiently. **Readiness is not a complete message guarantee:** TCP may deliver only some bytes, and a writable socket may accept only part of a buffer.

##### 1. `fd_set` — collection of watched descriptors
From `<sys/select.h>`, `fd_set` is an opaque descriptor-set type. Do not depend on its internal layout; use the macros:

```c
fd_set readfds;
FD_ZERO(&readfds);                   // Empty the set
FD_SET(server_fd, &readfds);        // Add a descriptor
FD_CLR(server_fd, &readfds);        // Remove a descriptor
int ready = FD_ISSET(server_fd, &readfds); // Nonzero if descriptor is in set
```

- `FD_ZERO(fd_set *set)`: Clear the entire set.
- `FD_SET(int fd, fd_set *set)`: Add a descriptor.
- `FD_CLR(int fd, fd_set *set)`: Remove a descriptor.
- `FD_ISSET(int fd, fd_set *set)`: Test membership; after `select()`, it indicates readiness.
- Descriptors must be nonnegative and **less than `FD_SETSIZE`**; exceeding that limit is unsafe. For many clients, prefer `poll()` or platform-specific `epoll`/`kqueue`.

##### 2. `struct timeval` — optional timeout
From `<sys/time.h>`:

```c
struct timeval {
    time_t      tv_sec;               // Whole seconds
    suseconds_t tv_usec;              // Additional microseconds (0–999999)
};
```

```c
struct timeval timeout = { .tv_sec = 5, .tv_usec = 0 }; // Wait up to 5 seconds
```

- Pass `NULL` to wait indefinitely.
- Pass `{0, 0}` to check without waiting.
- Reinitialize timeout before each call; some systems modify it.

##### 3. `select()` — wait for readiness

```c
#include <sys/select.h>
int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
```

- `nfds`: **Highest descriptor being watched + 1**, not the number of sockets.
- `readfds`: Set of descriptors to check for readable data or incoming connections.
- `writefds`: Set of descriptors to check for ability to write without blocking.
- `exceptfds`: Set of descriptors to check for exceptional conditions (not a general error set).
- `timeout`: Maximum waiting time, or `NULL` for unlimited waiting.
- Returns `> 0` for the number of ready descriptors, `0` for timeout, `-1` for error (including `EINTR` if interrupted).
- `select()` **modifies the sets**, leaving only ready descriptors; rebuild working sets before every call.

##### 4. Example: one thread, multiple TCP clients
This example assumes `server_fd` is a listening TCP socket. It handles readable sockets one at a time and uses a small fixed array; real servers must also handle partial sends and slow clients.

```c
#include <sys/select.h>
#include <sys/socket.h>
#include <unistd.h>
#include <stdio.h>

#define MAX_CLIENTS 100
int clients[MAX_CLIENTS];
for (int i = 0; i < MAX_CLIENTS; i++) clients[i] = -1;

while (1) {
    fd_set readfds;
    FD_ZERO(&readfds);
    FD_SET(server_fd, &readfds);
    int maxfd = server_fd;

    for (int i = 0; i < MAX_CLIENTS; i++) {
        if (clients[i] >= 0) {
            FD_SET(clients[i], &readfds);
            if (clients[i] > maxfd) maxfd = clients[i];
        }
    }

    int count = select(maxfd + 1, &readfds, NULL, NULL, NULL);
    if (count < 0) { perror("select"); continue; }

    if (FD_ISSET(server_fd, &readfds)) {
        int fd = accept(server_fd, NULL, NULL);
        if (fd >= 0) {
            int slot = -1;
            for (int i = 0; i < MAX_CLIENTS; i++) {
                if (clients[i] == -1) { slot = i; break; }
            }
            if (slot >= 0 && fd < FD_SETSIZE) clients[slot] = fd;
            else close(fd);                   // Full or descriptor too large
        }
    }

    for (int i = 0; i < MAX_CLIENTS; i++) {
        int fd = clients[i];
        if (fd < 0 || !FD_ISSET(fd, &readfds)) continue;
        char buffer[1024];
        ssize_t n = recv(fd, buffer, sizeof(buffer), 0);
        if (n > 0) {
            // Demonstration: one send may send fewer than n bytes
            if (send(fd, buffer, (size_t)n, 0) < 0) perror("send");
        } else {
            if (n < 0) perror("recv");
            close(fd);
            clients[i] = -1;
        }
    }
}
```

**Limitations:** This teaching loop assumes sockets are below `FD_SETSIZE` (check `server_fd` too), does not buffer incomplete writes, and does not cover all possible read errors. For a robust non-blocking event loop, track per-client input/output buffers and monitor write readiness when output remains.

##### 5. Real usage: one operator monitoring three connected sensors
Imagine three temperature sensors connected to one server. Instead of blocking on sensor A, the server places A, B, and C in an `fd_set` and waits for whichever becomes readable first.

```text
Sensors:   A ─────── data at 4 s ──────►
           B ─ data at 1 s ────────────►    ONE select() LOOP
           C ───────── data at 3 s ────►

Processing order: B (1 s) → C (3 s) → A (4 s)
```

Once A, B, and C are accepted sockets, the essential part of the loop looks like this:

```c
int clients[3] = {sensor_a_fd, sensor_b_fd, sensor_c_fd};
for (;;) {
    fd_set ready;
    FD_ZERO(&ready);
    int maxfd = -1;
    for (int i = 0; i < 3; i++) {
        if (clients[i] < 0) continue;   // Disconnected sensor
        FD_SET(clients[i], &ready);     // Assumes descriptor < FD_SETSIZE
        if (clients[i] > maxfd) maxfd = clients[i];
    }
    if (maxfd < 0) break;
    int count = select(maxfd + 1, &ready, NULL, NULL, NULL);
    if (count < 0) { if (errno == EINTR) continue; perror("select"); break; }
    for (int i = 0; i < 3; i++) {
        if (clients[i] < 0 || !FD_ISSET(clients[i], &ready)) continue;
        char data[128];
        ssize_t n = recv(clients[i], data, sizeof(data) - 1, 0);
        if (n > 0) { data[n] = '\0'; printf("Sensor %d: %s\n", i, data); }
        else if (n == 0) { close(clients[i]); clients[i] = -1; }
        else if (errno != EINTR) { perror("recv"); close(clients[i]); clients[i] = -1; }
    }
}
```

**Observable result:** `select()` sleeps until at least one socket is ready. Rebuild `fd_set` before every call because `select()` modifies it. This fragment assumes the three client descriptors have already been accepted; the complete server example above also watches the listening socket for new clients.


#### 2.4.6. HTML in Network Programming
A TCP server can send a **web page** to a browser by speaking **HTTP**, an application-layer protocol carried over TCP. HTML describes the page content; HTTP tells the browser how to interpret the bytes.

```text
Browser                               C TCP server
   │  GET / HTTP/1.1                     │
   ├────────────────────────────────────►│ recv() request
   │                                     │ Build HTML string in C
   │  HTTP/1.1 200 OK                    │
   │  Content-Type: text/html            │
   │  Content-Length: ...                │
   │  <html>...</html>                   │
   │◄────────────────────────────────────┤ send() response
   │  Render page                        │ close()
```

**Important difference:** Ordinary TCP echo servers send arbitrary bytes. An HTTP server must send a correctly formatted **HTTP response**: status line, headers, a blank line (`\r\n\r\n`), then the HTML body. The HTML below is constructed **inside the C source code**, with no external `.html` file.

##### 1. HTML text as a C string
Adjacent C string literals are concatenated by the compiler. `\n` means a newline in a C string; `\r\n` is the standard HTTP line ending.

```c
const char *html =
    "<!DOCTYPE html>\n"
    "<html>\n"
    "<head><meta charset=\"UTF-8\"><title>C Server</title></head>\n"
    "<body>\n"
    "  <h1>Hello from C!</h1>\n"
    "  <p>This HTML was written directly inside the C source.</p>\n"
    "</body>\n"
    "</html>\n";
```

- `<!DOCTYPE html>`: Tells the browser to use modern HTML parsing.
- `<html>`: Root of the page; `<head>` stores metadata; `<body>` holds visible content.
- `<h1>`: Heading; `<p>`: Paragraph.
- `\"`: Escaped double quote so it becomes part of the C string.

##### 2. `snprintf()` — format the HTTP response header
From `<stdio.h>`:

```c
int snprintf(char *str, size_t size, const char *format, ...);
```

- `str`: Destination character buffer.
- `size`: Capacity **including** the terminating `\0`.
- `format`: Format string; `%zu` prints a `size_t`, `%s` prints a string.
- `...`: Values inserted into the format string.
- Returns the number of characters that **would** have been written, excluding `\0`; a result `>= size` indicates truncation, and a negative result indicates an error.

```c
char header[256];
size_t html_len = strlen(html);                   // Body byte count, no '\0'
int n = snprintf(header, sizeof(header),
    "HTTP/1.1 200 OK\r\n"
    "Content-Type: text/html; charset=UTF-8\r\n"
    "Content-Length: %zu\r\n"
    "Connection: close\r\n"
    "\r\n", html_len);
if (n < 0 || (size_t)n >= sizeof(header)) {
    // Error or header did not fit
}
```

`strlen()` from `<string.h>` has syntax `size_t strlen(const char *s);` and counts bytes before the first `\0`. For this UTF-8 HTML string, `Content-Length` must be the number of **bytes**, not the number of visible characters.

##### 3. `send()` and HTTP response structure
The existing `send()` function from section 2.4.1 transmits the header and HTML body. HTTP over TCP is a byte stream: **one `send()` is not guaranteed to send every byte**. Use a loop to handle partial sends.

```c
#include <sys/socket.h>
#include <errno.h>

int send_all_bytes(int fd, const char *data, size_t len) {
    size_t sent = 0;
    while (sent < len) {
        ssize_t n = send(fd, data + sent, len - sent, 0);
        if (n > 0) sent += (size_t)n;
        else if (n < 0 && errno == EINTR) continue;
        else return -1;                          // Other error or no progress
    }
    return 0;
}
```

##### 4. Complete example: HTML HTTP server in C
Open `http://127.0.0.1:8080/` in a browser after compiling and running. This minimal teaching server handles **one request at a time**; it reads a request only to demonstrate the HTTP exchange, always responds with the same page, and closes the connection.

```c
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int send_all_bytes(int fd, const char *data, size_t len) {
    size_t sent = 0;
    while (sent < len) {
        ssize_t n = send(fd, data + sent, len - sent, 0);
        if (n > 0) sent += (size_t)n;
        else if (n < 0 && errno == EINTR) continue;
        else return -1;
    }
    return 0;
}

int main(void) {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd < 0) { perror("socket"); return 1; }

    struct sockaddr_in addr = {0};
    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = htonl(INADDR_LOOPBACK); // Only local browser
    addr.sin_port = htons(8080);

    if (bind(server_fd, (struct sockaddr *)&addr, sizeof(addr)) < 0 ||
        listen(server_fd, 10) < 0) {
        perror("bind/listen");
        close(server_fd);
        return 1;
    }
    printf("Open http://127.0.0.1:8080/\n");

    const char *html =
        "<!DOCTYPE html>\n"
        "<html><head><meta charset=\"UTF-8\"><title>C Server</title></head>\n"
        "<body><h1>Hello from C!</h1>\n"
        "<p>This page is built inside the C source code.</p></body></html>\n";

    while (1) {
        int client_fd = accept(server_fd, NULL, NULL);
        if (client_fd < 0) {
            if (errno == EINTR) continue;
            perror("accept");
            continue;
        }
        char request[4096];
        ssize_t n = recv(client_fd, request, sizeof(request) - 1, 0);
        if (n > 0) {
            request[n] = '\0';                  // For inspection/debugging
            char header[256];
            size_t html_len = strlen(html);
            int hlen = snprintf(header, sizeof(header),
                "HTTP/1.1 200 OK\r\n"
                "Content-Type: text/html; charset=UTF-8\r\n"
                "Content-Length: %zu\r\n"
                "Connection: close\r\n\r\n", html_len);
            if (hlen > 0 && (size_t)hlen < sizeof(header)) {
                if (send_all_bytes(client_fd, header, (size_t)hlen) == 0)
                    send_all_bytes(client_fd, html, html_len);
            }
        }
        close(client_fd);
    }
    close(server_fd);
    return 0;
}
```

**Run:** `gcc http_server.c -o http_server && ./http_server`. The browser sends an HTTP request; the C program sends back the header and in-source HTML. A complete HTTP implementation must parse requests, support different methods/paths, enforce size limits, and handle malformed or partial requests. This example is intended for local learning, not public deployment.

##### 5. Comparing the approaches

| Approach | Who handles multiple clients? | Does `recv()` wait? | Main trade-off |
|---|---|---|---|
| Blocking single-process | One execution flow | Yes | Simple, but idle client blocks progress |
| Non-blocking | One execution flow can switch sockets | No (returns `EAGAIN`) | Needs readiness and buffer management |
| Multi-processing | Separate child process per client | Can block within each child | Isolation, but higher resource overhead |
| Multi-threading | Separate thread per client | Can block within each thread | Shared memory, but synchronization needed |
| `select()` | One event loop watches many sockets | Call only when readable | Efficient waiting, but `FD_SETSIZE` limit |
| HTML over HTTP | Any of the above server architectures | Depends on chosen architecture | Adds HTTP headers and HTML response formatting |

##### 5. Real usage: inspect an actual HTTP request and response
Run the complete C HTTP server above, then in a second terminal run:

```bash
curl -v http://127.0.0.1:8080/
# To display response headers and HTML body together:
curl -i http://127.0.0.1:8080/
```

The browser (or `curl`) sends a request **as TCP bytes**. For example:

```http
GET / HTTP/1.1
Host: 127.0.0.1:8080
User-Agent: curl
Accept: */*

```

The server's `recv()` reads some of those bytes. It constructs the HTTP header and HTML body **inside the C program**, then calls `send()` (via `send_all_bytes()`) for each. The logical HTTP response looks like:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 176
Connection: close

<!DOCTYPE html>
<html><head><meta charset="UTF-8"><title>C Server</title></head>
<body><h1>Hello from C!</h1>
<p>This page is built inside the C source code.</p></body></html>
```

The `Content-Length` shown here is illustrative: **the running server calculates the actual byte count using `strlen(html)`**. On the wire, HTTP header lines end in `\r\n`, and a blank `\r\n` separates headers from HTML. TCP may split this response into multiple packets, but the browser reconstructs the byte stream.

```text
C source code                   TCP connection                   Browser
const char *html = ...
     │
strlen(html) → Content-Length
     │
send_all_bytes(header) ───────► HTTP status + headers ──────► Parse status / type
send_all_bytes(html)   ───────► HTML body bytes ────────────► Parse HTML / render page
close(client_fd)       ───────► End of connection
```

**Try changing the page:** Change `<h1>Hello from C!</h1>` to `<h1>Network Lab</h1>`, recompile and restart the server, then refresh the browser. The browser displays the new heading because the HTML is transmitted directly from the new C string. No `.html` file is read.

**HTTP detail:** `send()` does not send an HTML object; it sends raw bytes. `Content-Type: text/html` tells the browser to interpret the body as HTML, while `Content-Length` tells it how many body bytes to expect. The sample server is deliberately minimal: a production HTTP server must read and parse complete requests and handle errors and limits.


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
