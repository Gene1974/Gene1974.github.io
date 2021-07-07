# socket编程

#### sockaddr_in

```C++
#include <sys/socket.h>

struct sockaddr {  
     sa_family_t 		sin_family;	//地址族, 一般来说AF_INET（地址族）PF_INET（协议族）
　　  char 					sa_data[14]; //14字节，包含套接字中的目标地址和端口信息               
}; 

#include <netinet/in.h>
#或
#include <arpa/inet.h>

struct sockaddr_in { 
		short 					sin_family;		// Address family, 一般来说AF_INET（地址族）PF_INET（协议族）
		unsigned short 	sin_port; 		// Port number(网络数据格式,可以用htons()函数转换)
    struct in_addr 	sin_addr;			// IP address(in network byte order, Internet address）
    unsigned char 	sin_zero[8];	// 空白，用于对齐
}   
struct in_addr { 
    unsigned long 	s_addr;				// 
}
```



#### 主机字节序，网络字节序

主机字节序：
小端（Little-endian, LE）：地址低位存储值的低位 (x86, MIPS)
大端（Big-endian, BE）：地址低位存储值的高位

网络字节序：
均为大端，地址低位存储值的高位

存储0x01020304：

内存：             低位  --  高位
Little Endian: 04 03 02 01
Big Endian:    01 02 03 04 

常用函数：
inet_addr()  将字符串点数格式地址转化成无符号长整型（unsigned long s_addr s_addr;）
inet_aton()  将字符串点数格式地址转化成NBO
inet_ntoa ()   将NBO地址转化成字符串点数格式
htons()  "Host to Network Short"
htonl()  "Host to Network Long"
ntohs()  "Network to Host Short"
ntohl()  "Network to Host Long"