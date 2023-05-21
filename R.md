# Webserv:  42 School Project

<p align="center">
  <img width="100%" height="100%" src="test.png">
</p>

This project was completed as part of a 42 core curriculum. it was done with [SoukainaOuchen](https://github.com/souchen) & [LoubnaSoulima](https://github.com/lsoulima)
<br>

## About
The goal of the project is to build a C++ compatible HTTP web server from scratch.
The web server can handle HTTP GET, HEAD, POST, PUT, and DELETE Requests, and can serve static files from a specified root directory or dynamic content using CGI. It is also able to handle multiple client connections concurrently with the help of select().
<br>
---

# Usage
```bash
make
./webserv [Config File] ## leave empty to use the default configuration.
```

# The basics of an HTTP server:
HTTP (Hypertext Transfer Protocol) is an application layer protocol that is sent over a transport layer protocol(TCP, UDP, TLS).
    * Initially HTTP Client(web browser) sends an HTTP request to the HTTP Server.
    * Server processes the request received and sends HTTP response to the HTTP client.
The main purpose of a web server is to host web content and make it available to users over the internet.

<br>

# HTTP server Flow:
A basic HTTP web server consists of several components that work together to receive and process HTTP requests from clients and send back responses. When a client wants to communicate with a server, it performs the following steps:

1. Open a TCP connection: The TCP connection is used to send a request, or several, and receive an answer. The client may open a new connection, reuse an existing connection, or open several TCP connections to the servers.
2. Send an HTTP message: HTTP messages (before HTTP/2) are human-readable. With HTTP/2, these simple messages are encapsulated in frames, making them impossible to read directly, but the principle remains the same.
    __HTTP Message Format__
    ```
    start-line CRLF
    Headers CRLF
    CRLF(end of headers)
    [message-body]

    CRLF are Carriage Return and Line Feed (\r\n), which is just a new line.
    ```
HTTP Message can be either a request or response:
    __HTTP Request__
        ```
            GET / HTTP/1.1
            Host: developer.mozilla.org
            Accept-Language: fr
        ```
        1. The request line consists of three parts: the method, the path, and the HTTP version:
        - The method specifies the action that the client wants to perform, such as GET (to retrieve a resource) or POST (to submit data to the server):
        <br>
            __HTTP Methods__
            |Method|Description|Possible Body|
            |:----|----|:----:|
            |**`GET`** | Retrieve a specific resource or a collection of resources, should not affect the data/resource| No|
            |**`POST`** | Perform resource-specific processing on the request content| Yes|
            |**`DELETE`** | Removes target resource given by a URI| Yes|
            |**`PUT`** | Creates a new resource with data from message body, if resource already exist, update it with data in body | Yes|
            |**`HEAD`** | Same as GET, but do not transfer the response content  | No|
            <br>
            __GET__
            HTTP GET method is used to read (or retrieve) a representation of a resource. In case of success (or non-error), GET returns a representation of the resource in response body and HTTP response status code of 200 (OK). In an error case, it most often returns a 404 (NOT FOUND) or 400 (BAD REQUEST).
            <br>
            __POST__
            HTTP POST method is most often utilized to create new resources. On successful creation, HTTP response code 201 (Created) is returned.
            <br>
            __DELETE__
            HTTP DELETE is stright forward. It deletes a resource specified in URI. On successful deletion, it returns HTTP response status code 204 (No Content).
            <br>
        - The path or URI specifies the location of the resource on the server.
        - The HTTP version indicates the version of the HTTP protocol being used.
        2. Headers contain additional information about the request, such as the hostname of the server, and the type of browser being used.     
        <br>
    __HTTP Response__
        ```
        HTTP/1.1 200 OK
        Date: Sat, 09 Oct 2010 14:28:02 GMT
        Server: Apache
        Last-Modified: Tue, 01 Dec 2009 20:18:22 GMT
        ETag: "51142bc1-7449-479b075b2891b"
        Accept-Ranges: bytes
        Content-Length: 29769
        Content-Type: text/html
        <br>
        <!DOCTYPE html>… (here come the 29769 bytes of the requested web page)  //Message Body
        ```
        An HTTP response also consists of a status line, headers, and an optional message body:
        1. The status line consists of three parts: the HTTP version, the status code, and the reason phrase. 
        - The status code indicates the result of the request, such as 200 OK (successful) or 404 Not Found (resource not found). - The reason phrase is a short description of the status code. Following is a very brief summary of what a status code denotes:
            `1xx` indicates an informational message only
            `2xx` indicates success of some kind
            `3xx` redirects the client to another URL
            `4xx` indicates an error on the client's part
            `5xx` indicates an error on the server's part
        2. Headers contain additional information about the response, such as the type and size of the content being returned.
        3. The message body contains the actual content of the response, such as the HTML code for a webpage.

3. Read the response sent by the server, such as:
4. Close or reuse the connection for further requests.


# I/O Multiplexing & explanation:
1. We used select(),:
    - Include File(s): < sys/time.h>
    - Manual Section: 2
    - Summary: 
        int select(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout );

    - Return:
        Success: Number of ready file descriptors
        Failure:
            -1
        Sets errno:
            Yes



2. 
The I/O Multiplexing process in our web server is summarized in the flowchart below:
![I/O_Multiplexing](https://i1.ae/img/webserv/IO_M_BG.png)

![I/O_Multiplexing](select.gif)

- The socket() API returns a socket descriptor, which represents an endpoint. The statement also identifies that the INET (Internet Protocol) address family with the TCP transport (SOCK_STREAM) is used for this socket.
- After the socket descriptor is created, the bind() gets a unique name for the socket.
- The listen() allows the server to accept incoming client connections.
- The server uses the accept() API to accept an incoming connection request. The accept() API call blocks indefinitely, waiting for the incoming connection to arrive.
- The select() API allows the process to wait for an event to occur and to wake up the process when the event occurs. In this example, the select() API returns a number that represents the socket descriptors that are ready to be processed.
    0
        Indicates that the process times out. In this example, the timeout is set for 3 minutes.
    -1
        Indicates that the process has failed.
    1
        Indicates only one descriptor is ready to be processed. In this example, when a 1 is returned, the FD_ISSET and the subsequent socket calls complete only once.
    n
        Indicates that multiple descriptors are waiting to be processed. In this example, when an n is returned, the FD_ISSET and subsequent code loops and completes the requests in the order they are received by the server.

- The accept() and recv() APIs are completed when the EWOULDBLOCK is returned.
- The send() API echoes the data back to the client.
- The close() API closes any open socket descriptors.



# Ask if they use only one select() and how they've managed the server accept and the client read/write:

# the select() should be in the main loop and should check fd for read and write AT THE SAME TIME, if not please give a 0 and stop the evaluation:

# There should be only one read or one write per client per select(). Ask to show you the code that goes from the select() to the read and write of a client.

# Search for all read/recv/write/send on a socket and check that if an error returned the client is removed 

# Search for all read/recv/write/send and check if the returned value is well checked. (checking only -1 or 0 is not good you should check both)

# If a check of errno is done after read/recv/write/send. Please stop the evaluation and put a mark to O

# Writing or reading ANY file descriptor without going through the select() is strictly FORBIDDEN

# The project must compile without any re-link issue if not use the compile flag.
