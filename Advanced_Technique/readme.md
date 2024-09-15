# Advanced Topics

These are some advance technique and functionality, which quite used in real world network programming in C in Unix environment.

## Blocking :-

Here, "block" is same as "sleep", in technical terms. Many functions we have discussed so far, some of them are blocking function call. For example, recvfrom() will be in blocking state until some data arrives.

Some of the functions that blocks are :

* recvfrom()
* accept()
* socket()

If you want socket to be non-blocking :

```
#include <unistd.h>
#include <fcntl.h>

sockfd = socket(PF_INET, SOCK_STREAM, 0);
fcntl(sockfd, F_SETFL, O_NONBLOCK);
```

Setting a socket to non-blocking, leads to "polling" socket for information. If we try reading from non-blocking socket and there's no data there, it's not allowed to block it will return -1 and `errno` will be set to `EAGAIN` or `EWOULDBLOCK`.

> [!NOTE]
> The return value `EAGAIN` or `EWOULDBLOCK` is implementation dependent 
> and we should check both for portability.

However, this way of polling is not a great idea because it will be in constant busy-wait state and will be sucking out CPU time. However, the solution to this is using of `poll()`.

## poll() --- Synchronous I/O Multiplexing

* Purpose :

    It is used to monitor a *bunch* of sockets at once and then handle the ones that have data ready. 

> [!WARNING]
> *poll()* is horribly slow when it comes to a giant number of connections.
> In those circumstances, *libevent* will give better performance. It's a
> event notification library.

* Function Prototype :

    ```
    #include <poll.h>
    
    int poll(struct pollfd fds[], nfds_t nfds, int timeout);
    ```

    1. fds[] :

        Type : `struct pollfd`

        Value : Array of file descriptor stored in *struct pollfd*.

        Purpose : Each element store information about a particular fd, with *bitmap* of events we're interested in and also contain another buffer *bitmap* to stores the events that has occured.

    2. nfds :

        Type : `int`

        Value : Count of releavent fds.

        Purpose : To keep the count of releavent fds and discard the rest.

    3. timeout :

        Type : `int`

        Value : timeout in millisecond.

        Purpose : Time period to monitor this fds and let the process go to sleep, giving all the dirty work to the OS.

* Returns :

    returns the number of elements in the array that have had an event occur.

    After the poll() returns, check the `revents` field to see if POLLIN or POLLOUT is set, indicating the event occurred.

> [!NOTE]
> Structure of `struct pollfd`
>
> ```
> struct pollfd{
>   int fd;         // the socket descriptor
>   short events;   // bitmap of events we're interested in
>   short revents;  // when poll() returns, bitmap of events that occured.
> }
> ```
>
> The events field is the bitwise-OR of the following:
>
> POLLIN - Alert me when data is ready to recv() on this socket.
>
> POLLOUT - Alert me when I can send() data to this socket without blocking.

