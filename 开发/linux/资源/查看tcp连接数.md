# Linux下查看tcp连接数及状态

## 1. 查看tcp连接数及状态的语句：

#### 命令：



```shell
$ netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

#### 返回结果：



```text
TIME_WAIT 150
FIN_WAIT1 15
FIN_WAIT2 1
ESTABLISHED 55
SYN_RECV 21
CLOSING 2
LAST_ACK 4
```

## 2. TCP状态解析

**State:**

> **表TCP连接状态**

**ESTABLISHED:**

> **指TCP连接已建立，双方可以进行方向数据传递**

**CLOSE_WAIT:**

> **这种状态的含义其实是表示在等待关闭。当对方close一个SOCKET后发送FIN报文给自己，你系统毫无疑问地会回应一个ACK报文给对方，此时则进入到CLOSE_WAIT状态。接下来呢，实际上你真正需要考虑的事情是察看你是否还有数据发送给对方，如果没有的话， 那么你也就可以close 这个SOCKET，发送 FIN 报文给对方，也即关闭连接。所以你在CLOSE_WAIT 状态下，需要完成的事情是等待你去关闭连接。**

**LISTENING:**

> **指TCP正在监听端口，可以接受链接**

**TIME_WAIT:**

> **指连接已准备关闭。表示收到了对方的FIN报文，并发送出了ACK报文，就等2MSL后即可回到CLOSED可用状态了。如果FIN_WAIT_1状态下，收到了对方同时带FIN标志和ACK标志的报文时，可以直接进入到TIME_WAIT状态，而无须经过FIN_WAIT_2 状态。**

**FIN_WAIT_1:**

> **这个状态要好好解释一下，其实FIN_WAIT_1和 FIN_WAIT_2状态的真正含义都是表示等待对方的FIN报 文。而这两种状态的区别是：FIN_WAIT_1状态实际上是当SOCKET在ESTABLISHED状态时，它想主动关闭连接，向对方发送了FIN 报文，此时该SOCKET即进入到FIN_WAIT_1 状态。而当对方回应ACK 报文后，则进入到FIN_WAIT_2状态，当然在实际的正常情况 下，无论对方何种情况下，都应该马上回应ACK报文，所以FIN_WAIT_1状态一般是比较难见到的，而FIN_WAIT_2 状态还有时常常可以用 netstat看到。**

**FIN_WAIT_2：**

> **上面已经详细解释了这种状态，实际上FIN_WAIT_2 状态下的SOCKET，表示半连接，也即有一方要求close 连接，但另外还告诉对方，我暂时还有点数据需要传送给你，稍后再关闭连接。**

**LAST_ACK:**

> **是被动关闭一方在发送FIN报文后，最后等待对方的ACK报文。当收到ACK报文后，也即可以进入到CLOSED可用状态了**

**SYNC_RECEIVED:**

> **收到对方的连接建立请求,这个状态表示接受到了SYN报文，在正常情况下，这个状态是服务器端的SOCKET在建立TCP连接时的三次握手会话过程中的一个中间状态，很短暂，基本上用netstat你是很难看到这种状态的，除非你特意写了一个客户端测试程序，故意将三次TCP握手过程中最后一个ACK报文不予发送。因此这种状态时，当收到客户端的ACK报文后，它会进入到ESTABLISHED状态。**

**SYNC_SEND:**

> **已经主动发出连接建立请求。与SYN_RCVD遥想呼应，当客户端SOCKET执行CONNECT连接时，它首先发送SYN报文，因此也随即它会进入到了SYN_SENT状态，并等待服务端的发送三次握手中的第2个报文。**

上述状态的定义与TCP规则定义的TCP状态图一致，可以对照如下TCP建立与终止链接的状态转换图进行理解。

![img](https:////upload-images.jianshu.io/upload_images/14270006-4bfbb962a6560dc8.png?imageMogr2/auto-orient/strip|imageView2/2/w/562/format/webp)



![img](https:////upload-images.jianshu.io/upload_images/14270006-9665da56c38ab923.png?imageMogr2/auto-orient/strip|imageView2/2/w/746/format/webp)

建立TCP连接

![img](https:////upload-images.jianshu.io/upload_images/14270006-78fdc7e55309befc.png?imageMogr2/auto-orient/strip|imageView2/2/w/467/format/webp)

释放TCP连接
