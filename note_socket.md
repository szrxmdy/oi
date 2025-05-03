## 简介

[资源链接](https://beej.us/guide/bgnet/html)

基于 Unix 的一切皆文件的思想,socket 实际上是基于文件格式的,向操作系统传送该 socket 的文件描述符

### 数据报（datagram） 和 数据流（stream）

- stream 保证数据以发送的顺序到达目标点，需要先建立连接，保证数据无误传达（TCP），sock

- datagram 不保证数据到达，不需要建立连接，不保证到达顺序 （UDP）

- IP 是数据的路由协议

### 数据传输原理简介

数据包首先被应用程序通过 socket 发送给内核，内核套上 IP 协议发送给网卡，网卡再套上 MAC 协议最终发送，接受即一层层解析即可

即所谓的传输层，网络层，链路层 ...

值得注意的是，TCP/IP 协议规定网络传输使用大端模式（高位存储在低地址），但 Intel 使用小端模式，因此需要进行转换



## win 实战

[资源链接](https://learn.microsoft.com/zh-cn/windows/win32/winsock/getting-started-with-winsock)

### 头文件与链接库

注意到头文件只包含了定义，而具体的实现是在链接库（实际上就是编译后输出的目标文件）中的，在使用 Winsock2 时需要链接 WS2_32.dll ， 即在编译命令中添加 “ -lWs2_32 ”

由于 Windows.h 中包含了 Winsock , 会与 Winsock2 冲突，因此需要进行宏定义，具体的

```c++
#ifndef WIN32_LEAN_AND_MEAN
#define WIN32_LEAN_AND_MEAN
#endif

#include <windows.h>
#include <winsock2.h>
#include <ws2tcpip.h>
#include <iostream>
```



### 初始化（加载 dll)

在使用 Winsock 的所有函数之前，都需要进行初始化让系统将 winsock 的 dll 加载进入系统中

```c++
    // initialize winsock
    WSADATA wsadata;
    int iResult = WSAStartup(MAKEWORD(2,2), &wsadata);
    if (iResult != 0) {
        cout << "WSAStartup failed: " << iResult << endl;
        return 1;
    }
```



### 获取地址

一般使用 getaddrinfo（） 进行获取地址，原型为

```c++
int getaddrinfo(const char *nodename,const char *servname,const struct addrinfo *hints,struct addrinfo **res);
```

nodename 指向字符串，可以为 IPV4/6 地址或主机名或域名，null 或 localhost 为本机任意可用地址(存疑，待验证)

servname 用于指定端口，hints 指示希望得到的地址类型，res 返回结果，结果为链表形式，每一项为一个可能的可用地址

获取地址的程序实例

```c++
    addrinfo *result = NULL,*ptr = NULL,hints;
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_UNSPEC; // AF_INET or AF_INET6 to force version
    hints.ai_socktype = SOCK_STREAM; // TCP stream sockets
    hints.ai_protocol = IPPROTO_TCP; // TCP protocol

    #define DEFAULT_PORT "27015"

    // Resolve the server address and port
    {
        int iResult = getaddrinfo("localhost", DEFAULT_PORT, &hints, &result);
        if(iResult != 0) {
            cout << "getaddrinfo failed: " << iResult << endl;
            WSACleanup();
            return 1;
        }
    }

```

[addrinfo 的结构](https://learn.microsoft.com/zh-cn/windows/win32/api/ws2def/ns-ws2def-addrinfoa),ad_next 指示链表的下一项，空则为 NULL , 获得地址与端口存在 ai_addr 中

```c++
struct addrinfo {
  int ai_flags;
  int ai_family;
  int ai_socktype;
  int ai_protocol;
  size_t ai_addrlen;
  char *ai_canonname;
  struct sockaddr *ai_addr;
  struct addrinfo *ai_next;
}
```

由于 addrinfo 为链表的形式，因此释放空间需要使用

```c++
freeaddrinfo(addrinfo *ptr);
```

#### 地址的表示方式

**注意** ： 处于调用时的方便与代码通用性考虑，sockaddr 并不会严格解释为 sockaddr 的结构，而是会根据使用的协议族等上下文被解释为不同的结构，即使其定义为 sockaddr 类型。 [微软的解释](https://learn.microsoft.com/zh-cn/windows/win32/winsock/sockaddr-2)

- sockaddr_in 使用于 ipv4 协议

  ```c++
  struct sockaddr_in {
          short   sin_family;
          u_short sin_port;
          struct  in_addr sin_addr;
          char    sin_zero[8];
  };
  ```

- sockaddr_in6 适用于 ipv6 协议

  ```c++
  struct sockaddr_in6 {
          short   sin6_family;
          u_short sin6_port;
          u_long  sin6_flowinfo;
          struct  in6_addr sin6_addr;
          u_long  sin6_scope_id;
  };
  ```

- sockaddr_storage 同时适用于上述两种协议，且有足够大的保留空间用于未来的拓展，**被微软建议使用**

  ```c++
  struct sockaddr_storage {
    short   ss_family;
    char    __ss_pad1[_SS_PAD1SIZE];
    __int64 __ss_align;
    char    __ss_pad2[_SS_PAD2SIZE];
  }
  ```

### 创建套接字

```c++
ptr = result;
// Create a SOCKET for connecting to server
SOCKET Connectsocket = socket(ptr->ai_family, ptr->ai_socktype, ptr->ai_protocol);
if (Connectsocket == INVALID_SOCKET) {
    cout << "Error at socket(): " << WSAGetLastError() << endl;
    freeaddrinfo(result); // free the addrinfo structure
    WSACleanup();
    return 1;
}
```

result 即 getaddrinfo 返回的结果，传入 地址组，类型，协议 即可创建 socket

### 套接字绑定

将套接字绑定到某个端口上，默认情况下一个端口只能绑定一个套接字，如何绑定更多待填坑

如果不绑定系统会随机分配一个空闲端口给该套接字

```c++
int iResult = bind(listenSocket, ptr->ai_addr, (int)ptr->ai_addrlen);
if (iResult == SOCKET_ERROR) {
    cout << "bind failed: " << WSAGetLastError() << endl;
    closesocket(listenSocket);
    listenSocket = INVALID_SOCKET;
    return 1;
}
freeaddrinfo(result); // free the addrinfo structure
```

### 监听与接受连接

监听 （listen）与 接受连接（accept）通常是一起出现的，

listen 会将连接请求放入指定套接字的连接队列中，而 accept 会取出指定套接字连接队列的第一个连接到某个套接字中

监听代码 ：

```c++
// 进行监听
int iResult = listen(listenSocket, SOMAXCONN); // SOMAXCONN 由操作系统指定合理连接队列容量，自己指定可选 200-65535
if (iResult == SOCKET_ERROR) {
    cout << "listen failed: " << WSAGetLastError() << endl;
    closesocket(listenSocket);
    WSACleanup();
    return 1;
}
```

接受连接代码 ：

```c++
    sockaddr_storage clientAddr; int clientAddrLen = sizeof(clientAddr);
    SOCKET clientsocket = INVALID_SOCKET;
    clientsocket = accept(listenSocket, (sockaddr*)&clientAddr, &clientAddrLen);
    if(clientsocket == INVALID_SOCKET) {
        cout << "accept failed: " << WSAGetLastError() << endl;
        closesocket(listenSocket);
        WSACleanup();
        return 1;
    }
    closesocket(listenSocket);

    cout << clientAddrLen << endl;
    cout << ((sockaddr_in6*)&clientAddr)->sin6_port << endl; // 客户端的端口
```

注意阻塞实在 accept 阶段进行的，即如果待连接队列为空，则 accept 会造成阻塞

```c++
SOCKET WSAAPI accept(
  [in]      SOCKET   s,
  [out]     sockaddr *addr,
  [in, out] int      *addrlen
);
```

addr 用于返回客户端的地址信息，addrlen 输入 addr 的大小，返回地址的实际长度，

特别的，accept 后两个参数若为 NULL 则不会返回客户端的地址

###  连接

connect 用于由客户端发起连接请求，

```c++
// Connect to server
int iResult = connect(Connectsocket, ptr->ai_addr, (int)ptr->ai_addrlen);
if (iResult == SOCKET_ERROR) {
    closesocket(Connectsocket);
    Connectsocket = INVALID_SOCKET;
}
// 实际上此处应当尝试 addrinfo链表中的下一个地址 , 即 ptr = ptr -> ai_next
// 但在这里我们只尝试第一个地址
freeaddrinfo(result); // free the addrinfo structure
if (Connectsocket == INVALID_SOCKET) {
    cout << "Unable to connect to server!" << endl;
    WSACleanup();
    return 1;
}
```

### 发送和接收数据

#### [发送](https://learn.microsoft.com/zh-cn/windows/win32/api/winsock2/nf-winsock2-send)

```c++
int WSAAPI send( // 返回发送的字节数或 socketerror
  [in] SOCKET     s, 
  [in] const char *buf, // 发送内容起始地址
  [in] int        len, // 发送长度
  [in] int        flags // 为特殊调用方式，一般为 0 即可
);
```

例程 ：

```c++
const char *sendbuf = "this is a test";
int iResult;
iResult = send(Connectsocket, sendbuf, (int)strlen(sendbuf), 0);
if (iResult == SOCKET_ERROR) {
    cout << "send failed: " << WSAGetLastError() << endl;
    closesocket(Connectsocket);
    WSACleanup();
    return 1;
}
```

如果没有传输缓冲区，send 将会在发送完前处于阻塞状态

#### [接收](https://learn.microsoft.com/zh-cn/windows/win32/api/winsock/nf-winsock-recv)

```c++
int recv( // 返回收到的字节数，0 说明连接关闭，socketerror 说明出错
  [in]  SOCKET s,
  [out] char   *buf, // 接收地址
  [in]  int    len, // 接收缓冲区大小
  [in]  int    flags
);
```

例程 ：

```c++
#define DEFAULT_BUFLEN 512
int recvbuflen = DEFAULT_BUFLEN;
char recvbuf[DEFAULT_BUFLEN];

do {
    iResult = recv(clientsocket, recvbuf, recvbuflen, 0);
    if(iResult > 0) {
        cout << "Bytes received: " << iResult << endl;
        cout << "recvbuf: " << recvbuf << endl;
    } else if(iResult == 0) {
        cout << "Connection closed" << endl;
    } else {
        cout << "recv failed: " << WSAGetLastError() << endl;
        closesocket(clientsocket);
        WSACleanup();
        return 1;
    }
} while(iResult > 0);
```

### 关闭

```c++
iResult = shutdown(clientsocket, SD_SEND); // 先关闭发送端
if(iResult == SOCKET_ERROR) {
    cout << "shutdown failed: " << WSAGetLastError() << endl;
    closesocket(clientsocket);
    WSACleanup();
    return 1;
}

closesocket(clientsocket); // 关闭套接字
WSACleanup(); // 释放 dll 的使用
```

### 代码展示

#### 客户端

```c++
#ifndef WIN32_LEAN_AND_MEAN
#define WIN32_LEAN_AND_MEAN
#endif

#include <windows.h>
#include <winsock2.h>
#include <ws2tcpip.h>
#include <iostream>

using namespace std;
int main() {

    {
        WSADATA wsadata;
        // initialize winsock
        int iResult = WSAStartup(MAKEWORD(2,2), &wsadata);
        if (iResult != 0) {
            cout << "WSAStartup failed: " << iResult << endl;
            return 1;
        }
    }

    addrinfo *result = NULL,*ptr = NULL,hints;
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_UNSPEC; // AF_INET or AF_INET6 to force version
    hints.ai_socktype = SOCK_STREAM; // TCP stream sockets
    hints.ai_protocol = IPPROTO_TCP; // TCP protocol

    #define DEFAULT_PORT "27115"

    // Resolve the server address and port
    {
        int iResult = getaddrinfo("localhost", DEFAULT_PORT, &hints, &result);
        if(iResult != 0) {
            cout << "getaddrinfo failed: " << iResult << endl;
            WSACleanup();
            return 1;
        }
    }

    ptr = result;
    // Create a SOCKET for connecting to server
    SOCKET Connectsocket = socket(ptr->ai_family, ptr->ai_socktype, ptr->ai_protocol);
    if (Connectsocket == INVALID_SOCKET) {
        cout << "Error at socket(): " << WSAGetLastError() << endl;
        freeaddrinfo(result); // free the addrinfo structure
        WSACleanup();
        return 1;
    }

    {
        // Connect to server
        int iResult = connect(Connectsocket, ptr->ai_addr, (int)ptr->ai_addrlen);
        if (iResult == SOCKET_ERROR) {
            closesocket(Connectsocket);
            Connectsocket = INVALID_SOCKET;
        }
        // 实际上应当尝试 addrinfo链表中的下一个地址 , 即 ptr = ptr -> ai_next
        // 但在这里我们只尝试第一个地址
        freeaddrinfo(result); // free the addrinfo structure
        if (Connectsocket == INVALID_SOCKET) {
            cout << "Unable to connect to server!" << endl;
            WSACleanup();
            return 1;
        }
    }

    #define DEFAULT_BUFLEN 512
    const char *sendbuf = "this is a test";
    char recvbuf[DEFAULT_BUFLEN];
    int recvbuflen = DEFAULT_BUFLEN;

    int iResult;

    // Send an initial buffer
    iResult = send(Connectsocket, sendbuf, (int)strlen(sendbuf), 0);
    if (iResult == SOCKET_ERROR) {
        cout << "send failed: " << WSAGetLastError() << endl;
        closesocket(Connectsocket);
        WSACleanup();
        return 1;
    }

    cout << "Bytes Sent: " << iResult << endl;

    // shutdown SD_SEND 只关闭了发送端，仍然能接收
    iResult = shutdown(Connectsocket, SD_SEND); 
    if (iResult == SOCKET_ERROR) {
        cout << "shutdown failed: " << WSAGetLastError() << endl;
        closesocket(Connectsocket);
        WSACleanup();
        return 1;
    }

    do {
        iResult = recv(Connectsocket, recvbuf, recvbuflen, 0);
        if(iResult > 0) {
            cout << "Bytes received: " << iResult << endl;
            recvbuf[iResult] = '\0'; // null-terminate the string
            cout << recvbuf << endl;
        } else if (iResult == 0) {
            cout << "Connection closed" << endl;
        } else {
            cout << "recv failed: " << WSAGetLastError() << endl;
        }
    } while(iResult > 0);

    closesocket(Connectsocket);
    WSACleanup();
    cout << "Client closed" << endl;

    return 0;
}
```

#### 服务端

```c++
#ifndef WIN32_LEAN_AND_MEAN
#define WIN32_LEAN_AND_MEAN
#endif

#include <windows.h>
#include <winsock2.h>
#include <ws2tcpip.h>
#include <iostream>

using namespace std;
int main() {

    {
        // initialize winsock
        WSADATA wsadata;
        int iResult = WSAStartup(MAKEWORD(2,2), &wsadata);
        if (iResult != 0) {
            cout << "WSAStartup failed: " << iResult << endl;
            return 1;
        }
    }

    #define DEFAULT_PORT "27115"

    addrinfo *result = NULL,*ptr = NULL,hints;
    
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM; // TCP stream sockets
    hints.ai_protocol = IPPROTO_TCP; // TCP protocol
    // hints.ai_flags = AI_PASSIVE; // For wildcard IP address

    {
        int iResult = getaddrinfo(NULL, DEFAULT_PORT, &hints, &result);
        if(iResult != 0) {
            cout << "getaddrinfo failed: " << iResult << endl;
            WSACleanup();
            return 1;
        }
    }

    ptr = result;
    SOCKET listenSocket = socket(ptr->ai_family,ptr->ai_socktype,ptr->ai_protocol);
    if (listenSocket == INVALID_SOCKET) {
        cout << "Error at socket(): " << WSAGetLastError() << endl;
        freeaddrinfo(result); // free the addrinfo structure
        WSACleanup();
        return 1;
    }
    
    {
        int iResult = bind(listenSocket, ptr->ai_addr, (int)ptr->ai_addrlen);
        if (iResult == SOCKET_ERROR) {
            cout << "bind failed: " << WSAGetLastError() << endl;
            closesocket(listenSocket);
            listenSocket = INVALID_SOCKET;
            return 1;
        }
        freeaddrinfo(result); // free the addrinfo structure
    }

    {
        int iResult = listen(listenSocket, SOMAXCONN);
        if (iResult == SOCKET_ERROR) {
            cout << "listen failed: " << WSAGetLastError() << endl;
            closesocket(listenSocket);
            WSACleanup();
            return 1;
        }
    }

    SOCKET clientsocket = INVALID_SOCKET;
    clientsocket = accept(listenSocket, NULL, NULL);
    if(clientsocket == INVALID_SOCKET) {
        cout << "accept failed: " << WSAGetLastError() << endl;
        closesocket(listenSocket);
        WSACleanup();
        return 1;
    }
    closesocket(listenSocket);

    #define DEFAULT_BUFLEN 512
    int iResult,iSendResult;
    int recvbuflen = DEFAULT_BUFLEN;
    char recvbuf[DEFAULT_BUFLEN];
    
    do {
        iResult = recv(clientsocket, recvbuf, recvbuflen, 0);
        if(iResult > 0) {
            cout << "Bytes received: " << iResult << endl;
            cout << "recvbuf: " << recvbuf << endl;
        } else if(iResult == 0) {
            cout << "Connection closed" << endl;
        } else {
            cout << "recv failed: " << WSAGetLastError() << endl;
            closesocket(clientsocket);
            WSACleanup();
            return 1;
        }
    } while(iResult > 0);

    iResult = shutdown(clientsocket, SD_SEND); // 关闭发送端
    if(iResult == SOCKET_ERROR) {
        cout << "shutdown failed: " << WSAGetLastError() << endl;
        closesocket(clientsocket);
        WSACleanup();
        return 1;
    }

    closesocket(clientsocket);
    WSACleanup(); // clean up winsock
    cout << "Server closed" << endl;

    return 0;
}
```

