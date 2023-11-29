# socket相关API
```c
1. int socket(int domain, int type, int protocol);
//domain:协议簇， 常用的有AF_INET(IPV4), af_inet6(ipv6)...
//type socket类型，常用的有SOCK_STREAM, SOCK_DGRAM...
//protocol:指定协议，常用的有IPPROTO_TCP, IPPROTO_UDP...
//返回一个套接字描述符
1.1 struct sockaddr_in {
	sa_family_t    sin_family;
	in_port_t      sin_port;
	struct in_addr sin_addr;
}
//其中
struct in_addr{
	uint32_t s_addr;
}
//这一结构体用来存储socket套接字的地址信息，分别包含了协议簇和端口号以及IP地址的绑定；
//在使用任一套接字来操作socket编程时，必须明确该结构体内部的地址；
2. int bind(itn sockfd, const struct sockaddr *addr, socklen_t addrlen);
//用来绑定套接字和对应的套接字地址，这样可以直接使用套接字描述符来操作
//成功返回0，失败返回-1
3. int lisnten(int sockfd, int backlog);
//服务器侧用来监听，backlog代表可同时监听的客户端个数；
4. int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
//建立连接，请求向指定的套接字地址发起连接
5. int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
//服务器端接收tcp请求，该函数将会返回一个监听到的客户端套接字描述符
6. recv/send
//类似于read/write,它们的前三个参数相同，但recv和send多了第四个参数，默认是0；
```
# 双机通信demo (compiling by gcc)
```C
//server.c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define SERVER_PORT 8888

int main(int argc, char *argv[])
{
    int serverSocket;
    int clientSocket;

    struct sockaddr_in serverAddr;
    struct sockaddr_in clientAddr;

    int addrLen = sizeof(clientAddr);
    char sendBuf[1024];
    char recBuf[1024];
    int recDataNum;

    //---创建套接字---
    if ((serverSocket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)) < 0)
    {
        perror("socket create error.");
        exit(1);
    }
    
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(SERVER_PORT);
    serverAddr.sin_addr.s_addr = htonl(INADDR_ANY);

    //---绑定需要监听的IP和PORT---
    if (bind(serverSocket, (struct sockaddr*)&serverAddr, sizeof(serverAddr)) < 0)
    {
        perror("bind error.");
        exit(1);
    }

    //---监听---
    if (listen(serverSocket, 5) < 0)
    {
        perror("listen error.");
        exit(1);
    }

    //---接受和发送消息
    printf("正在监听的窗口:%d\n", SERVER_PORT);

    if ((clientSocket = accept(serverSocket, (struct sockaddr*)&clientSocket, (socklen_t*)&addrLen)) < 0)
    {
        perror("accept error.");
        exit(1);
    }

    printf("等待客户端消息...\n");

    printf("IP is: %s\n", inet_ntoa(clientAddr.sin_addr));
    printf("Port is: %d\n", htons(clientAddr.sin_port));

    while (1)
    {
        recBuf[0] = 0;
        if ((recDataNum = recv(clientSocket, recBuf, 1024, 0)) < 0)
        {
            continue;
        }

        recBuf[recDataNum] = '\0';
        if (strcmp(recBuf, "quit") == 0)
            break;
        printf("收到消息：%s\n", recBuf);
        printf("发送消息:");
        gets(sendBuf);
        if (send(clientSocket, sendBuf, strlen(sendBuf), 0) < 0)
        {
            perror("send error.");
            exit(1);
        }

        if (strcmp(sendBuf, "quit") == 0)
        {
            break;
        }
    }

    close(clientSocket);
    close(serverSocket);

    exit(0);
}
```
```c
//client.c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define SERVER_PORT 8888 //服务器端口号

int main(int argc, char *argv[])
{
    int serverSocket; // 创建套接字描述符
    struct sockaddr_in serverAddr; //创建服务器socket

    char sendBuf[1024];
    char recBuf[1024]; //接受和发送缓冲区

    int recDataNum; //接收字节个数


    //-------创建套接字--------
    if ((serverSocket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)) < 0)
    {
        perror("socket create error.");
        exit(1);
    }

    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(SERVER_PORT);

    //指定ip地址
    serverAddr.sin_addr.s_addr = inet_addr(argv[1]);

    //------------连接到服务器-----------
    if (connect(serverSocket, (struct sockaddr*)&serverAddr, sizeof(serverAddr)) < 0)
    {
        perror("connect error.");
        exit(1);
    }
    printf("已成功连接到主机...\n");

    //-----------发送和接收消息-----------
    while (1)
    {
        printf("发送消息：");
        gets(sendBuf);
        //printf("\n");
        if (send(serverSocket, sendBuf, strlen(sendBuf), 0) < 0)
        {
            perror("send error.");
            exit(1);
        }

        if (strcmp(sendBuf, "quit") == 0)
        {
            close(serverSocket);
            exit(0);
        }

        recBuf[0] = 0;
        recDataNum = recv(serverSocket, recBuf, sizeof(recBuf), 0);
        recBuf[recDataNum] = '\0';
        printf("接收消息%s\n", recBuf);
    }

    exit(0);
}
```