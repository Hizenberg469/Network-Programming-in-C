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

    
