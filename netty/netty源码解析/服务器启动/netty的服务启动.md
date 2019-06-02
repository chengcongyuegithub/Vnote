# netty的服务启动
## 两个问题
服务段的Socket在哪儿初始化?
accept在哪儿执行?

## part1
* 创建服务端的Channel
```
bind()-------->initAndRigister()--------->newChannel()
```
```
通过反射NioServerSocketChanel的内容
```
```
 public NioServerSocketChannel() {
        this(newSocket(DEFAULT_SELECTOR_PROVIDER));
    }
```
```
    private static final SelectorProvider DEFAULT_SELECTOR_PROVIDER = SelectorProvider.provider();
```
```
public NioServerSocketChannel(ServerSocketChannel channel) {
        super(null, channel, SelectionKey.OP_ACCEPT);
        config = new NioServerSocketChannelConfig(this, javaChannel().socket());
    }
```
```
protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
        super(parent);
        this.ch = ch;
        this.readInterestOp = readInterestOp;
        try {
            ch.configureBlocking(false);
```
```
 protected AbstractChannel(Channel parent) {
        this.parent = parent;
        id = newId();
        unsafe = newUnsafe();
        pipeline = newChannelPipeline();
    }
```
* 初始化服务端的Channel
```
option和Channel的Attr自定义的属性
```
```
childOption和childAttrs
```
```
配置pipline 自定义的handler
```
```
添加连接器Acceptor
```
* 注册selector
```
ChannelFuture regFuture = config().group().register(channel);
```
```
public ChannelFuture register(Channel channel) {
        return register(new DefaultChannelPromise(channel, this));
    }
```
```
public ChannelFuture register(final ChannelPromise promise) {
        ObjectUtil.checkNotNull(promise, "promise");
        promise.channel().unsafe().register(this, promise);
        return promise;
    }
```
```
           AbstractChannel.this.eventLoop = eventLoop;
           if (eventLoop.inEventLoop()) {
                register0(promise);
            } else {
                try {
                    eventLoop.execute(new Runnable() {
                        @Override
                        public void run() {
                            register0(promise);
                        }
                    });
                } catch (Throwable t) {
                    logger.warn(
                            "Force-closing a channel whose registration task was not accepted by an event loop: {}",
                            AbstractChannel.this, t);
                    closeForcibly();
                    closeFuture.setClosed();
                    safeSetFailure(promise, t);
                }
            }
        }
```
```
protected void doRegister() throws Exception {
        boolean selected = false;
        for (;;) {
            try {
                // javaChannel注册unwrappedSelector(selector),没有感兴趣的事件
                selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), 0, this);
                return;
            } catch (CancelledKeyException e) {
                if (!selected) {
                    // Force the Selector to select now as the "canceled" SelectionKey may still be
                    // cached and not removed because no Select.select(..) operation was called yet.
                    eventLoop().selectNow();
                    selected = true;
                } else {
                    // We forced a select operation on the selector before but the SelectionKey is still cached
                    // for whatever reason. JDK bug ?
                    throw e;
                }
            }
        }
    }
```
```
pipeline.invokeHandlerAddedIfNeeded();

                safeSetSuccess(promise);
                pipeline.fireChannelRegistered();
```
上面就已经将服务器端的channel绑定到Selector上面去了,但是这个时候没有感兴趣的事件.selector也开始轮寻.
同时在完成上面的创建和注册之后会调用invokeHandlerAddedIfNeeded和fireChannelRegistered,就是handler中的两个方法

* 端口绑定
在这儿把感兴趣的事件注册上去是accept