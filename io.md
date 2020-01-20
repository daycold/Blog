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


