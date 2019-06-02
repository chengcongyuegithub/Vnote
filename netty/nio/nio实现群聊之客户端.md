# nio实现群聊之客户端
```
public class NioClient {

    public static void main(String[] args) {

        try {
            //每一个客户端都要一个selector对象 
            SocketChannel socketChannel = SocketChannel.open();
            socketChannel.configureBlocking(false);

            Selector selector = Selector.open();
            //注册到选择器中
            socketChannel.register(selector,SelectionKey.OP_CONNECT);
            //连接到服务器
            socketChannel.connect(new InetSocketAddress("127.0.0.1",8899));

            while(true)
            {   
                //阻塞
                selector.select();
                //当返回一系列的事件的时候
                Set<SelectionKey> keySet = selector.selectedKeys();
                for(SelectionKey selectionKey:keySet)
                {   
                    //判断类型
                    //如果是自己的类型
                    if(selectionKey.isConnectable())
                    {   
                        //就获取到客户端
                        SocketChannel client=(SocketChannel)selectionKey.channel();
                        // 
                        if(client.isConnectionPending())
                        {   
                            //如果连接成功
                            client.finishConnect();
                            //创建缓存
                            ByteBuffer writeBuffer =ByteBuffer.allocate(1024);
                            //读入到buffer中
                            writeBuffer.put((LocalDateTime.now()+" connect ok!").getBytes());
                            //读写转换
                            writeBuffer.flip();
                            //客户端写到服务端
                            client.write(writeBuffer);
                            //开启另一个线程,键盘 
                            ExecutorService executorService= Executors.newSingleThreadExecutor(Executors.defaultThreadFactory());
                            executorService.submit(()->{

                                while(true)
                                {
                                    try {
                                        writeBuffer.clear();
                                        InputStreamReader input=new InputStreamReader(System.in);
                                        BufferedReader br = new BufferedReader(input);
                                        String sendMessage=br.readLine();
                                        writeBuffer.put(sendMessage.getBytes());
                                        writeBuffer.flip();
                                        client.write(writeBuffer);
                                    }catch (Exception e)
                                    {
                                        e.printStackTrace();
                                    }
                                }
                            });
                        }
                        //将客户端的read注册到selector中去
                        client.register(selector,SelectionKey.OP_READ);
                    }else if(selectionKey.isReadable())
                    {   //如果是读的事件
                        SocketChannel client=(SocketChannel)selectionKey.channel();
                        ByteBuffer readBuffer=ByteBuffer.allocate(1024);
                        int count=client.read(readBuffer);
                        if(count>0)
                        {
                            String receivedMessage=new String(readBuffer.array(),0,count);
                            System.out.println(receivedMessage);
                        }
                    }

                }
                //set清空
                keySet.clear();
            }
        }catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```