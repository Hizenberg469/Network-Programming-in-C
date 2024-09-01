# Network Programming in C

Quick reference to `struct`(s) and apis' used for networking applicaton in C language.

Let's first discuss about the important `struct` and then how we use apis' to establish networking functionality.

## struct(s) used are :-

* **struct** addrinfo -

  ```
    struct addrinfo {
        int             ai_flags;       // AI_PASSIVE, AI_CANONNAME, etc.
        int             ai_family;      // AF_INET, AF_INET6, AF_UNSPEC
        int             ai_socktype;    // SOCK_STREAM, SOCK_DRAM
        int             ai_protocol;    // use 0 for "any"
        size_t          ai_addrlen;     // size of ai_addr in bytes
        struct scokaddr *addr;          // struct sockaddr_in or _in6
        char            *ai_canonname;  // full canonical hostname
        struct addrinfo *ai_next;       // linked list, next node
    };
  ```

    1. ai_flags :

        Type : `int`

        Value : `AI_PASSIVE`, `AI_CANONNAME`.

        Purpose : Help determine the processing path for `getaddrinfo()`.

        * For ex : `AI_PASSIVE` suggest the `getaddrinfo()` to get the address of the host machine the program is running on.

    2. ai_family :

        Type : `int`

        Value : `AF_INET (IPv4)`, `AF_INET6 (IPv6)` or `AF_UNSPEC`.

        Purpose : Specify which type of IP address (in terms of representation) does it hold.

    3. ai_socktype :

        Type : `int`

        Value : `SOCK_STREAM`, `SOCK_DRAM`.

        Purpose : Specify what kind of socket we are interested in using.

    4. ai_protocol :

        Type : `int`

        Value : use 0 for "any".

        Purpose : What kind of protocol we are intending to use.

    5. ai_addrlen :

        Type : `size_t`

        Value : size of ai_addr in bytes.

        Purpose : What is the size of struct sockaddr pointer holds.

    6. addr :

        Type : `struct sockaddr *`

        Value : address in form of `struct sockaddr`.

        Purpose : Holding the actual address.

    7. ai_cannonname :

        Type : `char *`

        *I don't know what's it's used for.*

    8. ai_next :

        Type : `struct addrinfo *`

        Value : address of next `struct addrinfo`.

        Purpose : Intending to have a linked list.

---

* **struct** sockaddr -

    ```
        struct sockaddr{
            unsigned short   sa_family;     // Address family, AF_XXX
            char             sa_data[14];   // 14 bytes of protocol address
        };
    ```

    1. sa_family :

        Type : `unsigned short`

        Value : `AF_XXX`

        Purpose : Specify the address family type.

    2. sa_data :

        Type : `char`

        Value : IP address in for raw byte data.

        Purpose : Buffer for storing the actual address in *Network Byte Order*.

> [!IMPORTANT]
> Using this structure and hard coding every thing on it is very tedious. Use `struct sockaddr_in` for IPv4 or `struct sockaddr_in6` for IPv6.
---

* **struct** sockaddr_in -
  
  ```
    struct sockaddr_in{
        short int           sin_family;   // Address family, AF_INET
        unsigned short int  sin_port;     // Port number
        struct in_addr      sin_addr;     // Internet address
        unsigned char       sin_zero[8];  // Same size as struct sockaddr
    };
  ```

    1. sin_family :

        Type : `short int`

        Value : `AF_INET`.

        Purpose : Specifying the IP address belongs to IPv4.

    2. sin_port :

        Type : `unsigned short int`

        Value : Port number.

        Purpose : Stores the port number which will be used by the created socket.

    3. sin_addr :

        Type : `struct in_addr`

        Value : Internet address

        Purpose : Act as a buffer to store Internet address in raw byte.

    4. sin_zero :

        Type : `unsigned char`

        Value : All element set to zero.

        Purpose : Used for padding to match the size of `struct sockaddr`.


* **struct** in_addr -

    ```
        struct in_addr{
            uint32_t s_addr;
        };
    ```

    1. s_addr :

        Type : `uint32_t`

        Value : IP address in raw byte in Network Byte Order.

        Purpose : Act as buffer to store IP address.

---

* **struct** sockaddr_in6 -
  
    ```
        struct sockaddr_in6{
            u_int16_t      sin6_family;  // address family, AF_INET6
            u_int16_t      sin6_port;    // port number, Network Byte Order
            u_int32_t      sin6_flowinfo;// IPv6 flow information
            struct in6_addr sin6_addr;   // IPv6 address
            u_int32_t      sin6_scope_id;// Scope ID
        };
    ```

> `sin6_family`, `sin6_port` and `sin6_addr` are similar to the corresponding fields of `struct sockaddr_in6`.

* **struct** in6_addr -

    ```
        struct in6_addr{
            unsigned char s6_addr[16];  //IPv6 address
        };
    ```

    1. s6_addr :

        Type : `unsigned char array`

        Value : IPv6 address in raw byte.

        Purpose : Act as a buffer to store IPv6 address.

---

* **struct** sockaddr_storage -

> [!IMPORTANT]
> Because `sockaddr_in6` is larger in size than `sockaddr_in` and we may not know which to use when, we can use `sockaddr_storage` for storing any of the two and type-cast to their respective type.

    ```
        struct sockaddr_storage{
            sa_family_t     ss_family;  // address family

            /*
            * all this is padding, implementation specific, ignore it.
            */

            char        __ss_pad1[_SS_PAD1SIZE];
            int64_t     __ss_align;
            char        __ss_pad2[_SS_PAD2SIZE];
        };
    ```

1. ss_family :

    Type : `sa_family_t`

    Value : `AF_INET` or `AF_INET6`.

    Purpose : To distinguish address family it contains.