# 简介

## 概述
1.编写计算机网络通信程序，首先要确定这些程序通信所用协议。

## 简单时间获取客户程序
```cpp
#include	"unp.h"

int
main(int argc, char **argv)
{
	int					sockfd, n;
	char				recvline[MAXLINE + 1];
	
    //网络套接字地址结构
    struct sockaddr_in	servaddr;

	if (argc != 2)
		err_quit("usage: a.out <IPaddress>");

    /*
     socket函数创建一个AF_INET字节流(SOCK_STREAM)套接字
    */
	if ( (sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
		err_sys("socket error");

    /*
    bzero将结构体清零
	地址族置为AF_INET
	端口号置为13  
    */
	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port   = htons(13);	/* daytime server */
	if (inet_pton(AF_INET, argv[1], &servaddr.sin_addr) <= 0)
		err_quit("inet_pton error for %s", argv[1]);

    /*
	 与servaddr指定的服务器建立一个TCP连接
	*/
	if (connect(sockfd, (SA *) &servaddr, sizeof(servaddr)) < 0)
		err_sys("connect error");

	while ( (n = read(sockfd, recvline, MAXLINE)) > 0) {
		recvline[n] = 0;	/* null terminate */
		if (fputs(recvline, stdout) == EOF)
			err_sys("fputs error");
	}
	if (n < 0)
		err_sys("read error");

	exit(0);
}

```

## 协议无关性
上述程序的IPV6版本
```cpp
#include	"unp.h"

int
main(int argc, char **argv)
{
	int					sockfd, n;

	//修改一
	struct sockaddr_in6	servaddr;
	
	char				recvline[MAXLINE + 1];

	if (argc != 2)
		err_quit("usage: a.out <IPaddress>");

    //修改二
	if ( (sockfd = socket(AF_INET6, SOCK_STREAM, 0)) < 0)
		err_sys("socket error");

	bzero(&servaddr, sizeof(servaddr));
	
	//修改三
	servaddr.sin6_family = AF_INET6;
	//修改四
	servaddr.sin6_port   = htons(13);	/* daytime server */
	//修改五
	if (inet_pton(AF_INET6, argv[1], &servaddr.sin6_addr) <= 0)
		err_quit("inet_pton error for %s", argv[1]);

	if (connect(sockfd, (SA *) &servaddr, sizeof(servaddr)) < 0)
		err_sys("connect error");

	while ( (n = read(sockfd, recvline, MAXLINE)) > 0) {
		recvline[n] = 0;	/* null terminate */
		if (fputs(recvline, stdout) == EOF)
			err_sys("fputs error");
	}
	if (n < 0)
		err_sys("read error");

	exit(0);
}
```
 可以看出修改了5处程序代码。而更好的做法是编写协议无关的程序。

 ## 简单时间获取服务器程序
 ```cpp
#include	"unp.h"
#include	<time.h>

int
main(int argc, char **argv)
{
	int					listenfd, connfd;
	struct sockaddr_in	servaddr;
	char				buff[MAXLINE];
	time_t				ticks;

    //套接字的创建与客户端相同
	listenfd = Socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family      = AF_INET;
	//指定IP地址为INADDR_ANY，使得服务器进程可以在本机所有网卡ip地址上接受客户连接
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port        = htons(13);	/* daytime server */

	Bind(listenfd, (SA *) &servaddr, sizeof(servaddr));

    //调用listen函数将该套接字转换成一个监听套接字，使得来自客户的连接可以在该套接字上由内核接受
	Listen(listenfd, LISTENQ);

	for ( ; ; ) {
		//通常服务器进程在accept调用中被置于休眠状态，等待客户连接到达
		//返回值称作已连接描述符
		connfd = Accept(listenfd, (SA *) NULL, NULL);

        ticks = time(NULL);
        snprintf(buff, sizeof(buff), "%.24s\r\n", ctime(&ticks));
        Write(connfd, buff, strlen(buff));

		Close(connfd);
	}
}
 ```

 ## 测试用网络以及主机  
 1.大多数UNIX系统都提供了发现网络细节的两个基本命令：netstat和ifconfig
   * netstat -i 提供网络接口信息，还可以指定-n标志输出数值地址
   * netstat -r 展示路由表
   * 执行ifconfig可以获得每个网络接口的详细信息
   * 找出本地网络中众多主机IP地址的方法之一是针对上一步找到的本地接口的广播地址执行 ping -b 命令

  

