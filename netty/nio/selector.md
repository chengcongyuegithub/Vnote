# selector
## 例1
```
        int[] ports=new int[5];
        ports[0]=5000;
        ports[1]=5001;
        ports[2]=5002;
        ports[3]=5003;
        ports[4]=5004;
        
        //创建一个选择器
        Selector selector = Selector.open();
        for(int i=0;i<ports.length;i++)
        {   
            //服务端的通道
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
            //设置为非阻塞,否则会报错
            serverSocketChannel.configureBlocking(false);
            //通过每个端口的服务通道获取到serverSocket,因为服务端注册到选择器中是通过一个socketChannel完成的  
            ServerSocket serverSocket=serverSocketChannel.socket();
            //下面两句话就是完成端口绑定功能
            InetSocketAddress address=new InetSocketAddress(ports[i]);
            serverSocket.bind(address);
            //实现注册
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
            System.out.println("listening port: "+ports[i]);
        }
```
画一张图
![](_v_images/20190521152847606_1107407231.png =660x)
## 例2
```
在上面的5个serverSocketChannel都注册完成之后,现在就是等待连接了
while(true)
{           
            //如果没有符合条件的selectorKey就会一直阻塞,有符合条件的就返回
            int numbers=selector.select();
            System.out.println("numbers: "+numbers);
            //符合条件的集合返回
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            System.out.println("selectedKeys: "+selectionKeys);
            Iterator<SelectionKey> iter = selectionKeys.iterator();
            while(iter.hasNext())
            {
                SelectionKey selectionKey = iter.next();
                //如果事件的类型是Accept,肯定就是客户端和服务器建立联系
                if(selectionKey.isAcceptable())
                {    
                    //我们通过selectorKey的channel()就可以得到
                    ServerSocketChannel serverSocketChannel=(ServerSocketChannel)selectionKey.channel();
                    //如果有客户端A连接服务，执行select方法时，可以通过serverSocketChannel获取客户端A的socketChannel，并在        selector上注册socketChannel的OP_READ事件。
                    //建立连接的socketChannel
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    socketChannel.configureBlocking(false);
                    //socketChannel注册到选择器中
                    socketChannel.register(selector,SelectionKey.OP_READ);
                    //important
                    //当这个执行完毕之后,这个selectorKey就要移除了
                    iter.remove();
                }
                //如果是读入的事件
                else if(selectionKey.isReadable())
                {   
                    //读的SocketChannel
                    SocketChannel socketChannel=(SocketChannel)selectionKey.channel();
                    int bytesRead=0;
                    ByteBuffer byteBuffer=ByteBuffer.allocate(1024);
                    while(true)
                    {
                        byteBuffer.clear();
                        int read=socketChannel.read(byteBuffer);
                        if(read<=0)
                        {
                            break;
                        }
                        byteBuffer.flip();
                        //写回到写的channel
                        socketChannel.write(byteBuffer);
                        bytesRead+=read;
                    }
                    System.out.println("read: "+bytesRead+" from "+socketChannel);
                    iter.remove();
                }
            }
        }
```
![](_v_images/20190521154312901_2051955365.png =660x)