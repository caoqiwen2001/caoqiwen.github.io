---
title: NIO(二) 通道
date: 2018-02-01 16:59:05
tags:
  - Java  
categories:  
  - NIO  
type: tags
---
## NIO的通道


NIO的通道是IO基础上的创新，类似于原I/O中的流，它通过和缓冲区结合，可以使用最小的开销来访问操作系统本身的I/O服务。
目前已知的Channel的实现类有这些：
- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel  
来看一下FileChannel通道的继承关系图  

![image](http://note.youdao.com/yws/public/resource/16ff7aad6bffa41a21a5feb77e28578f/xmlnote/DDD513791CB04B5F9A74E3E353E37ACF/11253)  
先来看看Channel类代码的实现：

```
public interface Channel extends Closeable {

    /**
     * Tells whether or not this channel is open.
     *
     * @return <tt>true</tt> if, and only if, this channel is open
     */
    public boolean isOpen();

    /**
     * Closes this channel.
     *
     * <p> After a channel is closed, any further attempt to invoke I/O
     * operations upon it will cause a {@link ClosedChannelException} to be
     * thrown.
     *
     * <p> If this channel is already closed then invoking this method has no
     * effect.
     *
     * <p> This method may be invoked at any time.  If some other thread has
     * already invoked it, however, then another invocation will block until
     * the first invocation is complete, after which it will return without
     * effect. </p>
     *
     * @throws  IOException  If an I/O error occurs
     */
    public void close() throws IOException;

}
```
发现通道都是用接口实现的，然后只有两个方法，一个打开，一个关闭。至于为什么要用接口，对于不同的通道有不同的打开或者关闭操作。子接口ReadableByteChannel和WritableByteChannel的read和write方法都是基于ByteBuffer,基于字节来操作的。
再来看下ReadableByteChannel和WritableByteChannel的代码：

```
public interface ReadableByteChannel extends Channel {

    public int read(ByteBuffer dst) throws IOException;

}
```

```
public interface WritableByteChannel
    extends Channel
{

    public int write(ByteBuffer src) throws IOException;

}
```
通道通过继承ReadableByteChannel和WritableByteChannel使得可以进行读和写操作。

经典的读和写操作：

```
   private static void read(String filename) throws IOException {
        FileChannel fc = new FileInputStream(filename).getChannel();
        ByteBuffer buff = ByteBuffer.allocate(BSIZE);
        fc.read(buff);
        buff.flip();
        while (buff.hasRemaining()) {
            System.out.print((char) buff.get());
        }
        fc.close();
    }
```

写如果要附加到源文件末尾可以这样写：

```
  private static void addToEnd(String filename) throws IOException {
        FileChannel fc = new RandomAccessFile(filename, "rw").getChannel();
        fc.position(fc.size());
        fc.write(ByteBuffer.wrap("Some more ".getBytes()));
        fc.close();
    }
```

也可以这样子：通过设置append模式。

```
  /**
     * 文件追加到文件的末尾。
     * @param filename
     * @throws IOException
     */
     static  void addByOutput(String filename)throws  IOException{
         FileChannel fc=new FileOutputStream(filename,true).getChannel();
         fc.position(fc.size());
         fc.write(ByteBuffer.wrap("ni hao ".getBytes()));
         fc.close();
    }
```
####  FileChannel源码实现
##### read方法
下面重点看看FileChannel的源码：
FileChannel的read方法是通过FileChannelImpl来实现的：

```
   public int read(ByteBuffer var1) throws IOException {
        this.ensureOpen();
        if(!this.readable) {
            throw new NonReadableChannelException();
        } else {
            Object var2 = this.positionLock;
            synchronized(this.positionLock) {
                int var3 = 0;
                int var4 = -1;

                byte var5;
                try {
                    this.begin();
                    var4 = this.threads.add();
                    if(this.isOpen()) {
                        do {
                            var3 = IOUtil.read(this.fd, var1, -1L, this.nd);
                        } while(var3 == -3 && this.isOpen());

                        int var12 = IOStatus.normalize(var3);
                        return var12;
                    }

                    var5 = 0;
                } finally {
                    this.threads.remove(var4);
                    this.end(var3 > 0);

                    assert IOStatus.check(var3);

                }

                return var5;
            }
        }
    }
```
可以看到，read方法通过调用IOUtil.read方法来进行读的。

```
static int read(FileDescriptor var0, ByteBuffer var1, long var2, NativeDispatcher var4) throws IOException {
        if(var1.isReadOnly()) {
            throw new IllegalArgumentException("Read-only buffer");
        } else if(var1 instanceof DirectBuffer) {
            return readIntoNativeBuffer(var0, var1, var2, var4);
        } else {
            ByteBuffer var5 = Util.getTemporaryDirectBuffer(var1.remaining());

            int var7;
            try {
                int var6 = readIntoNativeBuffer(var0, var5, var2, var4);
                var5.flip();
                if(var6 > 0) {
                    var1.put(var5);
                }

                var7 = var6;
            } finally {
                Util.offerFirstTemporaryDirectBuffer(var5);
            }

            return var7;
        }
    }
```
在read方法中，会调用getTemporaryDirectBuffer方法申请DirectBuffer堆内存，然后通过readIntoNativeBuffer方法将数据读取到缓冲区var5，读取完之后在复制到var1。

```
 private static int readIntoNativeBuffer(FileDescriptor var0, ByteBuffer var1, long var2, NativeDispatcher var4) throws IOException {
        int var5 = var1.position();
        int var6 = var1.limit();

        assert var5 <= var6;

        int var7 = var5 <= var6?var6 - var5:0;
        if(var7 == 0) {
            return 0;
        } else {
            boolean var8 = false;
            int var9;
            if(var2 != -1L) {
                var9 = var4.pread(var0, ((DirectBuffer)var1).address() + (long)var5, var7, var2);
            } else {
                var9 = var4.read(var0, ((DirectBuffer)var1).address() + (long)var5, var7);
            }

            if(var9 > 0) {
                var1.position(var5 + var9);
            }

            return var9;
        }
    }
```
可以看到，底层是通过NativeDispatcher的read方法来实现的。
##### write方法
write方法和read方法的实现大体差不多：

```
public int write(ByteBuffer var1) throws IOException {
        this.ensureOpen();
        if(!this.writable) {
            throw new NonWritableChannelException();
        } else {
            Object var2 = this.positionLock;
            synchronized(this.positionLock) {
                int var3 = 0;
                int var4 = -1;

                byte var5;
                try {
                    this.begin();
                    var4 = this.threads.add();
                    if(this.isOpen()) {
                        do {
                            var3 = IOUtil.write(this.fd, var1, -1L, this.nd);
                        } while(var3 == -3 && this.isOpen());

                        int var12 = IOStatus.normalize(var3);
                        return var12;
                    }

                    var5 = 0;
                } finally {
                    this.threads.remove(var4);
                    this.end(var3 > 0);

                    assert IOStatus.check(var3);

                }

                return var5;
            }
        }
    }
```
通过调用IOUtil的write方法将buffer中的数据进行处理。


```
 static int write(FileDescriptor var0, ByteBuffer var1, long var2, NativeDispatcher var4) throws IOException {
        if(var1 instanceof DirectBuffer) {
            return writeFromNativeBuffer(var0, var1, var2, var4);
        } else {
            int var5 = var1.position();
            int var6 = var1.limit();

            assert var5 <= var6;

            int var7 = var5 <= var6?var6 - var5:0;
            ByteBuffer var8 = Util.getTemporaryDirectBuffer(var7);

            int var10;
            try {
                var8.put(var1);
                var8.flip();
                var1.position(var5);
                int var9 = writeFromNativeBuffer(var0, var8, var2, var4);
                if(var9 > 0) {
                    var1.position(var5 + var9);
                }

                var10 = var9;
            } finally {
                Util.offerFirstTemporaryDirectBuffer(var8);
            }

            return var10;
        }
    }
```
这里也是申请了一块DirectBuffer，将bytebuffer中数据放到var8中，调用writeFromNativeBuffer方法来实现写操作。

```
private static int writeFromNativeBuffer(FileDescriptor var0, ByteBuffer var1, long var2, NativeDispatcher var4) throws IOException {
        int var5 = var1.position();
        int var6 = var1.limit();

        assert var5 <= var6;

        int var7 = var5 <= var6?var6 - var5:0;
        boolean var8 = false;
        if(var7 == 0) {
            return 0;
        } else {
            int var9;
            if(var2 != -1L) {
                var9 = var4.pwrite(var0, ((DirectBuffer)var1).address() + (long)var5, var7, var2);
            } else {
                var9 = var4.write(var0, ((DirectBuffer)var1).address() + (long)var5, var7);
            }

            if(var9 > 0) {
                var1.position(var5 + var9);
            }

            return var9;
        }
    }
```
其底层都是通过NativeDispatcher类中的write方法来实现写操作。

### 总结
- read方法导致数据复制了两次。
- write方法也导致数据复制了两次。








