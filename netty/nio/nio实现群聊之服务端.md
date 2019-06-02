# nio实现群聊
```
public class NioServer {

    private static Map<String,SocketChannel> clientMap=new HashMap<>();

    public static void main(String[] args) throws Exception{
        //只需要一个serverSocketChannel,我们通过open这个静态方法得到
        ServerSocketChannel serverSocketChannel=ServerSocketChannel.open();
        serverSocketChannel.configureBlocking(false);//设置为异步
        ServerSocket serverSocket = serverSocketChannel.socket();//通过serverSocket进行绑定端口,所以服务器的端口就是8899
        serverSocket.bind(new InetSocketAddress(8899));
        
        //开启一个selector,整个程序里面只需要这一个选择器
        Selector selector = Selector.open();
        //注册serverSocketChannel
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        while(true)
        {
            try {
                //blocking
                selector.select();
                //符合条件的事件的集合
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                selectionKeys.forEach(selectionKey -> {
                    final SocketChannel client;
                    try {
                        //和服务器建立连接的
                        if(selectionKey.isAcceptable())
                        {   
                            //获取到被请求的serverSocket
                            ServerSocketChannel server=(ServerSocketChannel)selectionKey.channel();
                            //获取到请求的客户机
                            client=server.accept();
                            //设置为异步
                            client.configureBlocking(false);
                            //这个监听读的时间
                            client.register(selector,SelectionKey.OP_READ);
                            String key="{"+ UUID.randomUUID().toString()+"}";
                            //我们记录这个client
                            clientMap.put(key,client);
                        }
                        else if(selectionKey.isReadable())//如果是读事件
                        {   
                            //我们要获取到发起时间的客户机,也就是谁发的消息
                            client=(SocketChannel) selectionKey.channel();
                            ByteBuffer readBuffer=ByteBuffer.allocate(1024);
                            //读到应用程序
                            int count=client.read(readBuffer);
                            //如果读到了信息 
                            if(count>0)
                            {    
                                //这个时候我们要通过buffer的进行写,读写模式的切换 
                                readBuffer.flip();
                                //先指定编码
                                Charset charset=Charset.forName("utf-8");
                                //将接受的信息通过编码格式得到
                                String receivceMessage=String.valueOf(charset.decode(readBuffer).array());
                                //显示一个发起的人和要发送的信息
                                System.out.println(client+" : "+receivceMessage);
                                //然后我们就要开始发送消息了
                                String senderKey=null;
                                //找到发送人的uuid
                                for(Map.Entry<String,SocketChannel> entry:clientMap.entrySet())
                                {
                                    if(client==entry.getValue())
                                    {
                                        senderKey=entry.getKey();
                                        break;
                                    }
                                }
                                //然后广播发送
                                for(Map.Entry<String,SocketChannel> entry:clientMap.entrySet())
                                {
                                    //得到接受人的信息
                                    SocketChannel value=entry.getValue();
                                    ByteBuffer writeBuffer=ByteBuffer.allocate(1024);
                                    //将信息写入到缓冲区
                                    writeBuffer.put((senderKey+" : "+receivceMessage).getBytes());
                                    writeBuffer.flip();//转换
                                    value.write(writeBuffer);//写出
                                }
                            }
                        }
                    }catch (Exception e)
                    {
                        e.printStackTrace();
                    }

                });
                //important一定要clear();
                selectionKeys.clear();
            }catch (Exception e)
            {
                e.printStackTrace();
            }
        }
    }
}
```