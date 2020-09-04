> 近期学习NIO的时候，发现`ServerSocketChannel`有两种绑定端口的方式：

```java
// 方式1

InetSocketAddress inetSocketAddress = new InetSocketAddress(7000);
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(inetSocketAddress); 
SocketChannel scoketChannel = serverSocketChannel.accept();
```

```java
// 方式2

InetSocketAddress inetSocketAddress = new InetSocketAddress(7000);
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.bind(inetSocketAddress);
SocketChannel socketChannel = serverSocketChannel.accept();
```

两种方式唯一的区别在于`ServerSocketChannel`是否先获取了`ServerSoket`。

```java
// 方式1
serverSocketChannel.socket() // 先获取ServerScoket
    .bind(inetSocketAddress); // 再调用ServerScoket的bind方法
```

```java
// 方式2
// 直接调用ServerSocketChannel的实现类ServerSocketChannelImpl的bind方法
serverSocketChannel.bind(inetSocketAddress);
```

通过追源码的方式可以看到：

```java
// ServerScoket类的bind方法实现，JDK 1.4 引入
// backlog = 50
public void bind(SocketAddress endpoint, int backlog) throws IOException {
        if (isClosed())
            throw new SocketException("Socket is closed");
        if (!oldImpl && isBound())
            throw new SocketException("Already bound");
        if (endpoint == null)
            endpoint = new InetSocketAddress(0);
        if (!(endpoint instanceof InetSocketAddress))
            throw new IllegalArgumentException("Unsupported address type");
        InetSocketAddress epoint = (InetSocketAddress) endpoint;
        if (epoint.isUnresolved())
            throw new SocketException("Unresolved address");
        if (backlog < 1)
          backlog = 50;
        try {
            SecurityManager security = System.getSecurityManager();
            if (security != null)
                security.checkListen(epoint.getPort());
            getImpl().bind(epoint.getAddress(), epoint.getPort());
            getImpl().listen(backlog);
            bound = true;
        } catch(SecurityException e) {
            bound = false;
            throw e;
        } catch(IOException e) {
            bound = false;
            throw e;
        }
    }
```

```java
// ServerSocketChannelImpl类的bind方法实现，JDK 1.7 引入
// var2 = 0
public ServerSocketChannel bind(SocketAddress var1, int var2) throws IOException {
        Object var3 = this.lock;
        synchronized(this.lock) {
            if (!this.isOpen()) {
                throw new ClosedChannelException();
            } else if (this.isBound()) {
                throw new AlreadyBoundException();
            } else {
                InetSocketAddress var4 = var1 == null ? new InetSocketAddress(0) : Net.checkAddress(var1);
                SecurityManager var5 = System.getSecurityManager();
                if (var5 != null) {
                    var5.checkListen(var4.getPort());
                }

                NetHooks.beforeTcpBind(this.fd, var4.getAddress(), var4.getPort());
                Net.bind(this.fd, var4.getAddress(), var4.getPort());
                Net.listen(this.fd, var2 < 1 ? 50 : var2);
                Object var6 = this.stateLock;
                synchronized(this.stateLock) {
                    this.localAddress = Net.localAddress(this.fd);
                }

                return this;
            }
        }
    }
```

**结论**

最后可以发现两个实现方式大同小异，都是调用`JDK`原生的函数来实现绑定操作，`JDK1.7`以上的版本调用我们上文提到的`ServerSocketChannelImpl`的`bind`方法实现，`JDK1.7`以下的版本调用`ServerSocket`的`bind`方法实现，`bind`操作会将`channel`的`socket`绑定到本地地址，并配置`socket`以侦听连接。