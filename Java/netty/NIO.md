## NIO 概述

NIO全称为Non-blocking IO，其包含三大组件

### 1. Channel

Channel是双向的数据传输通道，不同于单向的InputStream和OutputStream，他们是单向的.

常见的Channel:

* FileChannel：用于文件传输
* DatagramChannel：用于UDP
* SocketChannel：用于TCP，客户端和服务端都能用
* ServerSocketChannel：用于TCP，专用于服务端

### 2. Buffer

Buffer就是存储数据的缓冲区。

常见的Buffer：

* ByteBuffer
  * MappedByteBuffer
  * DirectByteBuffer
  * HeapByteBuffer
* ShortBuffer
* IntBuffer

### 3. selector

TBD

## ByteBuffer

基本使用：

```java
public class TestByteBuffer
{
    public static void main( String[] args ) {
        try {
            // 1.创建channel和buffer
            FileChannel channel = new FileInputStream("D:\\Source Code\\JAVA\\Netty\\NIO-1\\BasicUse\\src\\main\\data.txt").getChannel();
            ByteBuffer buffer = ByteBuffer.allocate(10);

            while (true) {
                // 2. 读取数据到buffer
                int len = channel.read(buffer);

                // 3. 将buffer中的内容打印出来
                buffer.flip();      // 将buffer切换至读模式
                while (buffer.hasRemaining()) {
                    System.out.println((char) buffer.get());
                }
                buffer.clear();     // 将buffer切换为写模式

                // 4. 判断数据是否读取完
                if (len == -1) {
                    break;
                }
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

