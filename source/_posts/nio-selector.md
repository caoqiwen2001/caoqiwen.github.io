---
title: NIO(三) 选择器
date: 2018-02-02 16:15:20
tags:
  - Java  
categories:  
  - NIO  
type: tags
---
## 选择器
最终来聊一下选择器，与传统的IO相比，选择器减少了创建多个线程对服务器的开销，因为线程的资源太宝贵了，针对传统的BIO阻塞模式，需要创建一个线程去处理socket请求，使得主线程去读数据的时候不会被阻塞，如果针对成千上万的请求，我们系统马上就会陷入崩溃状态。  
因此，NIO采用选择器（Selector）和选择键（SelectionKey）这种形式来处理客户端的请求。
- 选择器（Selector）  
它管理者一堆注册通道的集合和他们的就绪状态。
- 选择键（SelectionKey）  
SelectionKey它封装了特定的通道与特定选择器的注册关系，当调用SelectableChannel.register()方式时会返回一个SelectionKey值。
接着，下面这段代码来测试服务端与客户端进行通讯：
#### 服务端代码

```
public class SelectorServer {
    private static int port = 1234;
    private static ByteBuffer buffer = ByteBuffer.allocate(1024);

    public static void main(String[] args) {
        try {
            ServerSocketChannel ssc = ServerSocketChannel.open();
            ServerSocket serverSocket = ssc.socket();
            serverSocket.bind(new InetSocketAddress(port));
            ssc.configureBlocking(false);
            Selector selector = Selector.open();
            ssc.register(selector, SelectionKey.OP_ACCEPT);
            while (true) {
                //这里还是会阻塞。只是在处理请求的时候不会再需要通过多线程去处理数据而已。
                int n = selector.select();
                // System.out.println("查看你会不会阻塞");
                if (n == 0) continue;
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                while (iterator.hasNext()) {
                    SelectionKey sk = iterator.next();
                    if (sk.isAcceptable()) {
                        ServerSocketChannel ssc1 = (ServerSocketChannel) sk.channel();
                        SocketChannel sc = ssc1.accept();
                        sc.configureBlocking(false);
                        sc.register(selector, SelectionKey.OP_READ);
                    } else if (sk.isReadable()) {
                        readData(sk);
                    }
                    iterator.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void readData(SelectionKey key) {
        SocketChannel sc = (SocketChannel) key.channel();
        buffer.clear();
        try {
            while (sc.read(buffer) > 0) {   //这里一直会去读通道的值
                buffer.flip();
                while (buffer.hasRemaining()) {
                    System.out.print((char) buffer.get());
                }
                System.out.println("");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
1、将ServerSocketChannel注册到注册到selector上。  
2、不断轮询，调用select方法，这个方法是阻塞的，客户端有连接过来时，阻塞完毕，之后会一直轮询，看selector上的通道有多少。  
3、获取SelectionKey中通道的集合，然后遍历所有的key，看是否有可接受的链接，是否有数据可以读取。  
4、操作完之后需要将该键移除，因为这次对话已经结束了，客户端的通道已经关闭，服务端也需要对应操作。  
在接受客户端请求的时候没有重新去开一个线程做请求处理。这里拿到对应的SocketChannel然后将其注册到selector上，下一次轮询就可以根据它的可读状态进行数据读操作。 在断点调试过程中，我发现SocketChannel的read方法不是阻塞的，即使客户端的链接释放了，服务端的SelectionKey一直处于isReadable状态，但由于buffer中没有可读的数据，就退出去继续轮询。

#### 客户端代码

```
public class SelectorClient {

    private static String str = "hello world";
    private static final int port = 1234;
    private static final String ip = "127.0.0.1";
    private static final int thread_count = 5;

    private static class NonBlockingClient extends Thread {
        @Override
        public void run() {
            try {
                SocketChannel socketChannel = SocketChannel.open();
                socketChannel.configureBlocking(false);
                socketChannel.connect(new InetSocketAddress(ip, port));
                while (!socketChannel.finishConnect()) {
                    System.out.println("同" + ip + "的连接正在建立，请稍等！");
                    Thread.sleep(10);
                }
                System.out.println("连接已经建立到ip+端口！，时间为:" + System.currentTimeMillis());
                String writeStr = str + this.getName();
                ByteBuffer buffer = ByteBuffer.allocate(writeStr.length());
                buffer.put(writeStr.getBytes());
                buffer.flip();
                socketChannel.write(buffer);
                buffer.clear();
                socketChannel.close();
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        NonBlockingClient[] nbts = new NonBlockingClient[thread_count];
        for (int i = 0; i < thread_count; i++) {
            nbts[i] = new NonBlockingClient();
        }
        for (int i = 0; i < thread_count; i++) {
            nbts[i].start();
        }
        for (int i = 0; i < thread_count; i++) {
            nbts[i].join();
        }

    }
}


```
客户端通过开启五个线程进行测试，每一个线程中启动一个SocketChannel连接到客户端。
### 总结
Selector可以简化用单线程同时管理多个可选择通道的实现，可以减少服务端多开线程的开销，但是它并不是完全的非阻塞I/O。这种实现方式值得好好研究。



