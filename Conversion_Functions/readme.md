## Functions' used are :-

### Functions for conversion between network and host byte order :

   1. `uint32_t htonl( uint32_t hostlong )`
       * Convert *hostlong* in **host byte order** to **network byte order**.
   
   2. `uint16_t htons( uint16_t hostshort )`
       * Convert *hostshort* in **host byte order** to **network byte order**.
  
   3. `uint32_t ntohl( uint32_t netlong )`
       * Convert *netlong* in **network byte order** to **host byte order**.
    
   4. `uint16_t ntohs( uint16_t netshort )`
       * Convert *netshort* in **network byte order** to **host byte order**.


### Functions for conversion between network representation to presentation representation :

Since the IP address are presentable in form of (for eg.) 194.10.11.5 for IPv4, but the actual representation in network is in raw byte form. So, for easy conversion between the two, apis are provided for such conversion.

1. `const char *inet_ntop(int af, const void* src, char* dst, socklen_t size)`

    Parameters :
    
    * af :

        Type : `int`

        Value : `AF_INET`, `AF_INET6`.

        Purpose : Used to specify type of address we are converting (IPv4 or IPv6).

    * src :

        Type : `const void *`

        Value : Buffer containing raw byte address in form.

    * dst :

        Type : `char *`

        Value : Buffer for storing the IP address in presentable form.

    * size :

        Type : `socklen_t`

        Value : `INET_ADDRSTRLEN` or `INET6_ADDRSTRLEN`.

        Purpose : Maximum size of buffer to store the IP address.

    Return type :

    * Type : `const char *`

    * Purpose : Pointer to *dst* if no error occured, otherwise returns null pointer and set global error variable `errno`.

2. `int inet_pton(int af, const char *src, void *dst)`

    **Parameter :**

    * af :
    
        Type : `int`

        Value : `AF_INET`, `AF_INET6`.

        Purpose : Used to specify type of address we are converting (IPv4 or IPv6).

    * src :

        Type : `const char *`

        Value : String to convert address in presentable form to network compatible form.

    * dst :

        Type : `void *`

        Value : empty buffer.

        Purpose : To store address in network compatible form.