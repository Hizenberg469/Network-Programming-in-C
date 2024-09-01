# System Calls :

Below are the system calls that can be used to access network functionality.

* **getaddrinfo()** :
  
  * Function prototye

    ```
    int getaddrinfo(const char *node,  // e.g. "www.example.com" or IP
                    const char *service, // e.g. "http" or port number
                    const struct addrinfo *hints,
                    struct addrinfo **res);
    ```

    1. node :

        Type : `const char *`