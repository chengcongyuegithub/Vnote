# Http请求
```
public class TestServer {

    public static void main(String[] args)throws Exception {
        //相当于是两个死循环
        //listening
        EventLoopGroup bossGroup=new NioEventLoopGroup();//这个用来接受消息
        //read
        EventLoopGroup workerGroup=new NioEventLoopGroup();//这个用来处理并转发消息

        try {
            //服务器的封装
            ServerBootstrap serverBootstrap=new ServerBootstrap();
            //封装两个死循环
            serverBootstrap.group(bossGroup,workerGroup)
            //childhandler就是确定workerGroup要做的事情 
            .channel(NioServerSocketChannel.class).childHandler(new TestServerInitializer());
            //绑定端口
            ChannelFuture channelFuture=serverBootstrap.bind(8899).sync();
            //关闭
            channelFuture.channel().closeFuture().sync();
        }finally {
            //停止两个死循环
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```
**TestServerInitializer**就是workerGroup的逻辑代码,就是这个代码中表示的就是对消息的处理
```
public class TestServerInitializer extends ChannelInitializer<SocketChannel>{

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline=ch.pipeline();
        pipeline.addLast("httpServerCodec",new HttpServerCodec());
        pipeline.addLast("testHttpServerHandler",new TestHttpServerHandler());
    }
}
```
TestHttpServerHandler是真正的逻辑代码
```
//重写SimpleChannelInboundHandler中的方法,处理的消息类型是HttpObject
public class TestHttpServerHandler extends SimpleChannelInboundHandler<HttpObject>{

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpObject msg) throws Exception {
        //msg就是请求的信息
        System.out.println(msg.getClass());
        System.out.println(ctx.channel().remoteAddress());
        Thread.sleep(8000);
        //判断如果是http类型的请求 
        if(msg instanceof HttpRequest)
        {   
            //强制转化
            HttpRequest httpRequest=(HttpRequest)msg;
            System.out.println("请求方法名：" + httpRequest.method().name());
            //获取请求的uri
            URI uri=new URI(httpRequest.uri());
            if("/favicon.ico".equals(uri.getPath()))
            {
                System.out.println("favicon.ico");
                return ;
            }
            //返回信息学
            ByteBuf content= Unpooled.copiedBuffer("hello world", CharsetUtil.UTF_8);
            FullHttpResponse response=new DefaultFullHttpResponse(HttpVersion.HTTP_1_1,HttpResponseStatus.OK,content);
            response.headers().set(HttpHeaderNames.CONTENT_TYPE,"text/plain");
            response.headers().set(HttpHeaderNames.CONTENT_LENGTH,content.readableBytes());

            //ctx发送
            ctx.writeAndFlush(response);
            //ctx关闭
            ctx.channel().close();
        }
    }
    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelRegistered!!!");
        super.channelRegistered(ctx);
    }

    @Override
    public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelUnRegistered!!!");
        super.channelUnregistered(ctx);
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelActive!!!");
        super.channelActive(ctx);
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelInactive!!!");
        super.channelInactive(ctx);
    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        System.out.println("handlerAdded!!!");
        super.handlerAdded(ctx);
    }
}
```