# System Calls

Below are the system calls that can be used to access network functionality.

* **getaddrinfo()** :
  
  * Purpose :

    It helps set up the structs, specifically **struct addrinfo**. It does all the dirty work like DNS and service name lookup and fill up the structs with all the relevant information based on the parameters passed.
  
  * Function prototye

    ```
    int getaddrinfo(const char *node,  // e.g. "www.example.com" or IP
                    const char *service, // e.g. "http" or port number
                    const struct addrinfo *hints,
                    struct addrinfo **res);
    ```

    1. node :

        Type : `const char *`

        Value : Domain name or IP in form of string.

        Purpose : To help find the host computer IP address for connection.

    2. service :

        Type : `const char *`

        Value : Service/Protocol or Port in form of string.

        Purpose : To help specify the port number in the host computer.

    3. hints :

        Type : `const struct addrinfo *`

        Value : Pointer to *struct addrinfo*.

        Purpose : To guide how it should fetch relevent data and store it to `struct addrinfo **res`.

    4. res :

        Type : `struct addrinfo **`

        Value : Pointer to linked list of *struct addrinfo \**

  * Return Type :

      Non-zero value in case of error or zero, otherwise.

    **Code Snippet**

    ```
    int status;
    struct addrinfo hints;
    struct addrinfo *servinfo; // will point to the results

    memset(&hints, 0, sizeof hints); // make sure the struct is 
                                     // garbage free
    
    
    hints.ai_family = AF_UNSPEC;     // don't care IPv4 or IPv6
    hints.ai_socktype = SOCK_STREAM; // TCP stream sockets
    hints.ai_flags = AI_PASSIVE;     // fill in my IP(my host) for me

    if ((status = getaddrinfo(NULL, "3490", &hints, &servinfo)) != 0){
        fprintf(stderr, "getaddrinfo erro: %s\n", gai_strerror(status));
        exit(1);
    }

    // servinfo now  points to a linked list of 1 or more struct 
    // addrinfos


    // ... do everything until you don't need servinfo anymore ...

    freeaddrinfo(servinfo); // free the linked list
    ```

* **socket() :**

  * Purpose :

    Assign a *file descriptor* for accessing OS resources to access network. Defines the type connection it would make to other host.

  * Function Prototype :

    ```
    #include <sys/types.h>
    #include <sys/socket.h>

    int socket(int domain, int type, int protocol);
    ```

    1. domain :

        Type : `int`

        Value : `PF_INET` or `PF_INET6`

        Purpose : Used to specify the Protocol family we would like to use.

    > [!IMPORTANT]
    > The value of `PF_INET` and `AF_INET` are same and they are used 
    > interchangably. **PF** stands for *Protocol family* and **AF** stands 
    > stands for *Address family*.
    
    2. type :

        Type : `int`

        Value : `SOCK_STREAM` or `SOCK_DGRAM`

        Purpose : Used to signify type of socket we want to create. 
                  (Connection or Connection-less).

    3. protocol :

        Type : `int`

        Purpose : Signifies either `TCP` or `UDP` protocol is being used for communication.

  * Return Type :

      Returns' an integer which signify an `fd` or -1 on error and global variable *errno* is set to the error's value .

  * **Code Snippet**

    ```
    int s;
    struct addrinfo hints, *res;

    // do the lookup
    // [pretend we already filled out the "hints" struct]
    getaddrinfo("www.example.com", "http", &hints, &res);

    // remember, do the error-checking on getaddrinfo(),
    // walk the "res" linked list for valid entries.

    s = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
    ```

* **bind() :**

  * Purpose :
  
    It's used for connecting socket fd with port of local machine.

  * Function prototype :

    ```
    #include <sys/types.h>
    #include <sys/socket.h>

    int bind(int sockfd, struct sockaddr *my_addr, int addrlen);
    ```

    1. sockfd :

        Type : `int`

        Value : integer value representing socket fd.

        Purpose : sockfd to bind to a port.

    2. my_addr :

        Type : `struct sockaddr *`

        Value : Holding port number information.

        Purpose : port number to bind to sockfd.

    3. addrlen :

        Type : `int`

        Value : length of the `struct sockaddr`.

  * Return Type :

    returns -1 on error and sets **errno** to the error's value.

  >[!IMPORTANT]
  > The below code snippet is w.r.t old way of using bind function.
  > The new way of doing the same is to use `getaddrinfo()` for ease.

  ```
  // !!! THIS IS THE OLD WAY !!!

  int sockfd;
  struct sockaddr_in my_addr;

  sockfd = socket(PF_INET, SOCK_STREAM, 0);

  my_addr.sin_family = AF_INET;
  my_addr.sin_port = htons(MYPORT); // short, network byte order
  my_addr.sin_addr.s_addr = inet_addr("10.12.110.57");
  memset(my_addr.sin_zero, '\0', sizeof my_addr.sin_zero);

  bind(sockfd, (struct sockaddr *)&my_addr, sizeof my_addr);
  ```

  >[!NOTE]
  > Sometimes binding to a port maybe fail due to its live bond with a free 
  > socket and can lead to error. To avoid this below code snippet can be 
  > userful.
  >
  > ```
  > int yes = 1;
  > // char yes = '1'; // Solaris people use this
  > 
  > // lose the pesky "Address already in use" error message
  > if( setsockopt(listener,SOL_SOCKET, 
  > SO_REUSEADDR, &yes, sizeof yes) ==  -1){
  >   perror("setsockopt");
  >   exit(1);
  > }
  > ```

* **connect() :**

  * Purpose :

    To connect to a remote host.

  * Function prototype :

    ```
    #include <sys/types.h>
    #include <sys/socket.h>

    int connect(int sockfd, struct sockaddr *serv_addr, int addrlen);
    ```

    1. sockfd :

        Type : `int`

        Value : socket fd, as returned by the `socket()` call.

    2. serv_addr :

        Type : `struct sockaddr *`

        Value : containing destination port and IP address.

    3. addrlen :

        Type : `int`

        Value : length `struct sockaddr` in bytes.

  * Return :

    It'll return -1 on error and set the variable `errno`.

  * Code snippet :

    ```
    struct addrinfo hints, *res;
    int sockfd;

    // first, load up address structs with getaddrinfo():

    memset(&hints, 0, sizeof hints);
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;

    getaddrinfo("www.example.com", "3490", &hints, &res);

    // make a socket:

    sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

    // connect!

    connect(sockfd, res->ai_addr, res->ai_addrlen);
    ```

* **listen() :**

  * Purpose :

    Connect to incoming connection from remote host computers.

  * Function prototype :

    ```
    int listen(int sockfd, int backlog);
    ```

    1. sockfd :

        Type : `int`

        Value : socket fd, as returned by the `socket()` call.

    2. backlog :

        Type : `int`

        Value : number of connection allowed on the incoming queue.

  * Return :

    It'll return -1 on error and set the variable `errno`.

  * Code Snippet :

    ```
    getaddrinfo();
    socket();
    bind();
    listen();
    /* accept() goes here */
    ```

* **accept() :**

  * Purpose :

    Accept connection request pending in the waiting queue.

  * Function Prototype :

    ```
    #include <sys/types.h>
    #include <sys/socket.h>

    int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
    ```

    1. sockfd :

        Types : `int`

        Value : *sockfd* is the listening socket descriptor.

    2. addr :

        Type : `struct sockaddr *`

        Value : pointer to a local `struct sockaddr_storage`. This  is where the information about the incoming connection will go(and with it we can determine which host is calling us from which port).

    3. addrlen :

        Type : `int`

        Value : equal to value sizeof(struct sockaddr_storage). accept() will not put more than that many bytes into addr. If it puts fewer in, it'll change the value of addrlen to reflect that.

  * Return value :

      returns -1 and set `errno` if error occurs.

  * Code Snippet :

    ```
    #include <string.h>
    #include <sys/types.h>
    #include <sys/socket.h>
    #include <netdb.h>

    #define MYPORT "3490" // the port users will be connecting to
    #define BACKLOG 10 // how many pending connections queue will hold

    int main(){

      struct sockaddr_storage their_addr;
      socklen_t addr_size;
      struct addrinfo hints, *res;
      int sockfd, new_fd;

      // !! don't forget your error checking for these calls !!

      // first, load up address structs with getaddrinfo();

      memset(&hints, 0, sizeof hints);
      hints.ai_family = AF_UNSPEC; // use IPv4 or IPv6, whichever
      hints.ai_socktype = SOCK_STREAM;
      hints.ai_flags = AI_PASSIVE; // fill in my IP for me

      getaddrinfo(NULL, MYPORT, &hints, &res);

      // make a socket, bind it, and listen on it:

      sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
      bind(sockfd, res->ai_addr, res->ai_addrlen);
      listen(sockfd, BACKLOG);

      // now accept an incoming connection:

      addr_size = sizeof their_addr;
      new_fd = accept(sockfd, (struct sockaddr *)&their_addr, &addr_size);

      // ready to communicate on socket descriptor new_fd!
    }
    ```

      >[!NOTE]
      > new_fd will be used for send() and recv() calls.

* **send() :**

  * Purpose :

    Used for communication between two connections. The communication can over stream sockets or connected datagram sockets.

  * Function Prototype :

    ```
    int send(int socfd, const void *msg, int len, int flags);
    ```

    1. sockfd :

        Type : `int`

        Value : *sockfd* is the socket descriptor you want to send data to(whether it's the one returned by socket() or the one you got with accept()).

    2. msg :

        Type : `const void *`

        Value : pointer to data we want to send in form of byte array.

    3. len :

        Type : `int`

        Value : Length of byte array.

    4. flag :

        Type : `int`

        Value : set to 0.

  * Return type:
      
      returns the number of bytes actually sent out - *this might be less than the number you told it to send!*. Again, -1 is returned on error, and `errno` is set to the error number.

  * Code Snippet :

    ```
    char *msg = "Beej was here!";
    int len, bytes_sent;

    len = strlen(msg);
    bytes_sent = send(sockfd, msg, len, 0);
    ```

* **recv() :**
  
  * Purpose :

    To receive data from the remote machine, which may attempt to send data.

  * Function prototype :

    ```
    int recv(int sockfd, void *buf, int len, int flags);
    ```

    1. sockfd :

        Type : `int`

        Value : Socket desriptor to read from.

    2. buf :

        Type : `void *`

        Value : buffer to read the information into.

    3. len :

        Type : `int`

        Value : maximum length of buffer specified.

    4. flags :

        Type : `int`

        Value : set to 0.

  * Return type :

    Number of bytes actually read into the buffer or -1 on error. recv() can return 0 when the remote side has closed the connection.

* **sendto() :**

  * Purpose :

    To send data to remote connection without forming any connection beforehand.

  * Function Prototype :

    ```
    int sendto(int sockfd, const void *msg, int len, unsigned int flags,
                  const struct sockaddr *to, socklen_t tolen);
    ```

    1. sockfd :

        Type : `int`

        Value : fd which maintain connection with remote machine.

    2. msg :

        Type : `const void*`

        Value : buffer which stores the data which need to be transferred.

    3. len :

        Type : `int`

        Value : Length of the buffer

    4. flags :

        Type : `int`

        Value : set it to zero

    5. to :

        Type : `struct sockaddr *`

        Value : Containing the address we want the send data to.

    6. tolen :

        Type : `socklen_t`

        Value : `int` deep down, is equal to sizeof(struct sockaddr_storage).

* **recvfrom() :**

  * Purpose :

    It receive data by remote machine which choose udp mode of communication.

  * Function Prototype :

    ```
    int recvfrom(int sockfd, void *buf, int len, unsigned int flags,
              struct sockaddr *from, int *fromlen);
    ```

    1. sockfd :

        Type : `int`

        Value : fd which maintain connection with remote machine.

    2. buf :

        Type : `void *`

        Value : buffer which will stores the data that would be received.

    3. len :

        Type : `int`

        Value : Length of the buffer

    4. flags :

        Type : `int`

        Value : set it to zero

    5. from :

        Type : `struct sockaddr *`

        Value : contains the address of sending machine.

    6. fromlen :

        Type : `int`

        Value : equal to sizeof(struct sockaddr_storage)

  * Returns :

      returns number of bytes received, or -1 on error and set `errno`.

* **close() :**

  * Purpose :

    To close the socket and prevent any further communication.

  * Function Prototype :
  
    ```
    close(sockfd);
    ```

* **shutdown() :**

  * Purpose :

      It gives more control over how the socket closes. It allows you to cut off communication in a certain direction, or both ways. If applied on unconnected datagram sockets, it will simply make socket unavailable to use.

  * Function Prototype :

    ```
    int shutdown(int sockfd, int how);
    ```

    1. sockfd :

        which socket to close.

    2. how :

        how can have following value :-

        * 0 - Further receives are disallowed.
        * 1 - Further sends are disallowed.
        * 2 - Further sends and receives are disallowed(like close()).

  * Returns:

    returns 0 on success or -1 on failure.

> [!NOTE]
> `shutdown()` doesn't actually close the file descriptor -- it just
> changes its usability. To free socket descriptor, you need to use
> `close()`.

* **getpeername() :**

  * Purpose :
      
      It tells us who is at the other end of a connected stream socket.

  * Function Prototype :

      ```
      #include <sys/socket.h>

      int getpeername(int sockfd, struct sockaddr *addr, int *addrlen);
      ```

    1. sockfd :

        Descriptor of connected stream socket.

    2. addr :

        Holds the address of the remote connection.

    3. addrlen :

        Equal to the sizeof(struct sockaddr).

  * Returns :
  
    Returns -1 on error and sets errno accordingly.

* **gethostname() :**

  * Purpose :

    Gets name of the computer that running our program.

  * Function Prototype :

    ```
    #include <unistd.h>

    int gethostname(char *hostname, size_t size);
    ```

    1. hostname :

        buffer to contain the hostname.

    2. size :

        size of the buffer.