# IO
## InputStream
### 申明方法
#### read()
抽象方法，在 FileInputStream 中有native实现, 大部分实现中通过其他的InputStream引用实现该方法
#### read(byte[])
直接调用 read(byte[], 0, length)
#### read(byte[], int, int)
在for循环中调用read()方法，将返回的int转成char存入参数的数组中，遇到-1时结束
#### skip(long)
在循环中调用n（n小于2048或者为2048的公倍数）次read方法，抛弃这些数据
#### available()
返回可以调用read或者skip的剩余字节数
#### close()
synchronized关键字，默认空方法
#### reset()
synchronized关键字，默认抛出IOException
回滚道上一次调用mark方法的位置（如果没有被mark过，抛出IOException或者回到最开始的位置，看具体实现），从而可以重新read()或skip()
#### mark(int)
synchronized关键字，默认空方法，标记当前位置
#### markSupported()
判断是否支持mark和reset方法，默认返回false
### FileInputStream
不支持mark-reset，其他方法使用native方法重写
持有一个 FileChannel 和 FileDescriptor

## OutputStream
### 申明方法
#### write(int)
抽象方法
#### write(byte[])
调用 write(byte[], 0, length)
#### write(write(byte[], int, int)
遍历调用 write(int)
#### flush()
空实现
#### close()
空实现

### FileOutputStream
使用 native 方法实现 write 方法，并重写同名方法；重写 close 方法
持有一个 FileChannel 和 FileDescriptor

## Reader
InputStream 得到 bytes，Reader 将使用编码规则得到 chars
### 申明方法
#### read(CharBuffer)
read(char[], 0, length)得到一个int (char)
CharBuffer.put(char[], 0, len)
#### read()
返回 read(new char[1], 0 ,1)[0] 或 -1
#### read(char[])
调用 read(char[], 0, length)
#### read(char[], int, int)
抽象方法
#### skip(long)
类似 InputStream 的skip，多次调用read实现跳过
#### ready()
默认返回false
流是否可读
#### markSupported()
默认返回false
#### mark()
默认抛出IOException
#### reset()
默认抛出IOException
#### close()
抽象方法
### StreamDecoder
持有Charset， CharsetDecoder，
ByteBuffer， InputStream， ReadableByteChannel
### InputStreamReader
持有一个StreamDecoder，并通过它实现read方法
## Writer
持有一个 char 数组
### 申明方法
#### write(int)
将参数放在持有数组的第一位，并调用
write(char[], 0, 1）
#### write(char[])
调用 write(char[], 0, length)
#### write(char[], int, int)
抽象方法
#### write(String)
调用 write(String, 0, length)
#### write(String, int, int)
截取 String的 substring(int, int)
调用 write(char[], 0, length)
#### append(CharSequence)
调用 write(String)
返回自身
#### append(CharSequence, int, int)
调用 write(String.substring(int, int))
返回自身
#### append(char)
调用 write(int) 返回自身
#### flush
抽象方法
#### close
### StreamEncoder
持有 Charset, CharsetEncoder, ByteBuffer, OutputStream, WritableByteChannel, CharBuff
### OutputStreamWriter
持有一个StreamEncoder，通过它实现write方法


## Socket通信流程
[转自](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html)
###建立通信链路
当客户端要与服务端通信，客户端首先要创建一个 Socket 实例，操作系统将为这个 Socket 实例分配一个没有被使用的本地端口号，并创建一个包含本地和远程地址和端口号的套接字数据结构，这个数据结构将一直保存在系统中直到这个连接关闭。在创建 Socket 实例的构造函数正确返回之前，将要进行 TCP 的三次握手协议，TCP 握手协议完成后，Socket 实例对象将创建完成，否则将抛出 IOException 错误。

与之对应的服务端将创建一个 ServerSocket 实例，ServerSocket 创建比较简单只要指定的端口号没有被占用，一般实例创建都会成功，同时操作系统也会为 ServerSocket 实例创建一个底层数据结构，这个数据结构中包含指定监听的端口号和包含监听地址的通配符，通常情况下都是“*”即监听所有地址。之后当调用 accept() 方法时，将进入阻塞状态，等待客户端的请求。当一个新的请求到来时，将为这个连接创建一个新的套接字数据结构，该套接字数据的信息包含的地址和端口信息正是请求源地址和端口。这个新创建的数据结构将会关联到 ServerSocket 实例的一个未完成的连接数据结构列表中，注意这时服务端与之对应的 Socket 实例并没有完成创建，而要等到与客户端的三次握手完成后，这个服务端的 Socket 实例才会返回，并将这个 Socket 实例对应的数据结构从未完成列表中移到已完成列表中。所以 ServerSocket 所关联的列表中每个数据结构，都代表与一个客户端的建立的 TCP 连接。

###数据传输
传输数据是我们建立连接的主要目的，如何通过 Socket 传输数据，下面将详细介绍。

当连接已经建立成功，服务端和客户端都会拥有一个 Socket 实例，每个 Socket 实例都有一个 InputStream 和 OutputStream，正是通过这两个对象来交换数据。同时我们也知道网络 I/O 都是以字节流传输的。当 Socket 对象创建时，操作系统将会为 InputStream 和 OutputStream 分别分配一定大小的缓冲区，数据的写入和读取都是通过这个缓存区完成的。写入端将数据写到 OutputStream 对应的 SendQ 队列中，当队列填满时，数据将被发送到另一端 InputStream 的 RecvQ 队列中，如果这时 RecvQ 已经满了，那么 OutputStream 的 write 方法将会阻塞直到 RecvQ 队列有足够的空间容纳 SendQ 发送的数据。值得特别注意的是，这个缓存区的大小以及写入端的速度和读取端的速度非常影响这个连接的数据传输效率，由于可能会发生阻塞，所以网络 I/O 与磁盘 I/O 在数据的写入和读取还要有一个协调的过程，如果两边同时传送数据时可能会产生死锁，在后面 NIO 部分将介绍避免这种情况。

### 流程概况
响应请求为例（Socket发送请求后）。
Socket创建一个InputStream并分配一定内存，监听目标端口和地址，进入阻塞。得到数据后将数据写入InputStream的内存中，配合编码集将字节转成字符（出现乱码的地方，也是耗时操作），以便使用。


## NIO



