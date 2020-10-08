# 基本套接字编程

## 套接字地址结构
1.IPV4套接字结构——也称作“网际套接字结构”，以sockaddr_in命名，定义在<netinet/in.h>中。
```cpp
struct in_addr
{
    in_addr_t s_addr;
};



struct sockaddr_in
{
    //linux中没有这一字段
    uint8_t sin_len;
    
    /*
     POSIX规范只需要这3个字段
    */
    sa_family_t sin_family;
    in_port_t sin_port;			/* Port number.  */
    struct in_addr sin_addr;		/* Internet address.  */

    
    char sin_zero[8];
};

```
注意：
  * IPv4地址和TCP/UDP端口号在套接字地址结构中是以网络字节序来存储的
  * sin_zero字段未曾使用 (linux只是将其用于内存对齐) ，不过我们总是将该字段置为0

   
2.通用套接字地址结构：当作为参数传递进入任何套接字函数时，套接字地址总是以指针来传递，所以该套接字函数必须能够处理来自所支持的任何协议族的套接字地址结构。
```

```
