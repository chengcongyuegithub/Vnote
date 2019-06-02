# bind()
```
 ChannelFuture channelFuture=serverBootstrap.bind(8899).sync();
```
```
 public ChannelFuture bind(int inetPort) {
        return bind(new InetSocketAddress(inetPort));
    }
```
```
 public ChannelFuture bind(SocketAddress localAddress) {
        validate();
        if (localAddress == null) {
            throw new NullPointerException("localAddress");
        }
        return doBind(localAddress);
    }
```
```
final ChannelFuture regFuture = initAndRegister();
```
```
 final ChannelFuture initAndRegister() {
        Channel channel = null;
        try {
            channel = channelFactory.newChannel();
            //这里的channelFactory就是带有反射功能的channelFactory,newChannel就是创建了channel的实例
            //创建的实例是在上面的语句中的NioServerSocketChannel
            init(channel);
```
我们创建NioServerSocketChannel的过程要仔细分析一下
```
public NioServerSocketChannel() {
        this(newSocket(DEFAULT_SELECTOR_PROVIDER));
    }
```
```
private static final SelectorProvider DEFAULT_SELECTOR_PROVIDER = SelectorProvider.provider();

private static ServerSocketChannel newSocket(SelectorProvider provider) {
        try {
            /**
             *  Use the {@link SelectorProvider} to open {@link SocketChannel} and so remove condition in
             *  {@link SelectorProvider#provider()} which is called by each ServerSocketChannel.open() otherwise.
             *
             *  See <a href="https://github.com/netty/netty/issues/2308">#2308</a>.
             */
            return provider.openServerSocketChannel();
        } catch (IOException e) {
            throw new ChannelException(
                    "Failed to open a server socket.", e);
        }
    }
    
public NioServerSocketChannel(ServerSocketChannel channel) {
        super(null, channel, SelectionKey.OP_ACCEPT);
        config = new NioServerSocketChannelConfig(this, javaChannel().socket());
    }
protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
        super(parent);
        this.ch = ch;
        this.readInterestOp = readInterestOp;//设置了感兴趣的值
        try {
            ch.configureBlocking(false);//设置了异步
```




然后就是 init(channel);
```
首先是对channel中的属性进行配置,一些option
ChannelPipeline p = channel.pipeline();//然后就是channel中的管道
        final EventLoopGroup currentChildGroup = childGroup;
        final ChannelHandler currentChildHandler = childHandler;
              赋值EventLoopGroup,ChannelHandler
```

```
 p.addLast(new ChannelInitializer<Channel>() {
            @Override
            public void initChannel(final Channel ch) throws Exception {
                final ChannelPipeline pipeline = ch.pipeline();//获取到管道
                ChannelHandler handler = config.handler();
                if (handler != null) {//就是childrenHandler之前的handler
                    pipeline.addLast(handler);
                }
                //将所有的信息设置进去
                ch.eventLoop().execute(new Runnable() {
                    @Override
                    public void run() {
                        //接收器
                        pipeline.addLast(new ServerBootstrapAcceptor(
                                ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                    }
                });
            }
        });
```