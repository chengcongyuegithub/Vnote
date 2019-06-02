# Scalable IO In Java
## 网络编程
```
读请求----->解码请求----->处理请求----->编码请求----->发送请求
```
其中每一步的花费是不同的
```
xml解析,file的传输,网页的生成.....
```
## 传统的服务设计
### 概述
![](_v_images/20190526103237766_1253075936.png =660x)
每一个handler都有自己的一个线程
代码的实现
```
class Server implement Runnable
{
    public void run()
    {
       try{
         ServerSocket ss=new ServerSocket(port); 
         while(true)
         {
             new Thread(new Handler(ss.accept)).start();
         }
       }catch(.....)
       {
       }
    }
    //这个就对应这上面的handler,在它的run方法中做的事情就是依次来执行read...... 
    static class Handler implements Runnable{
        final Socket socket;
        public Handler(Socket s){ socket=s; }
        public void run()
        {
           byte[] input=new byte[MAX_INPUT];
           socket.getInputStream().read(input);
           byte[] output=process(input);
           socket.getOutputStream().write(output);
        }
    } 
}
```
相当于Server一直都是处于监听状态,每一个连接要建立,就要创建一个线程new Thread(severSocket.accept()).start,每一个线程就对应这一个handler.
```
while(true)
{
   new Thread(new Handler(ss.accept)).start(); 
}
```
我们在Handler中做的事情就是图中的handler按顺序做下来的事情,就是一个顺序执行
```
           byte[] input=new byte[MAX_INPUT];
           socket.getInputStream().read(input);
           byte[] output=process(input);
           socket.getOutputStream().write(output);
```
### 实现的目标
* 在不断增加客户机的时候,能够优雅的降级
* 在不断增加资源的时候,不断的改善
* 更短的延迟,满足高峰期的要求,可调节的服务需求
```
(分治法)divide and conquer是实现这些目标的最好的方法
```
## 分治法
我们将过程分解为更小的任务,每一个任务都是在不阻塞的情况下进行的.我们来看一看上面的例子
handler中5个步骤都是同步执行的,这样CPU就有可能等待I/O操作.
**I/O的事件经常作为触发器服务**
NIO就能完成这些事情,不阻塞的读和写,分发事件对于这些I/O事件.
基于**事件驱动**的设计
### 事件驱动
* 更少的资源,我们不必为每一个客户端都开启一个线程
* 更少的花费,更少的上下文切换,更少的损失
* 但是分发就会很慢,大多都是绑定行为对于事件的.
* 事件驱动的实现更加困难,必须跟踪服务的状态
## Reactor模式
Reactor的工作模式是通过分发不同的handler来对IO事件进行应答
handler做的就是非阻塞的工作,将handler绑定到events,相当于是addListener
![](_v_images/20190526110803161_987415201.png =660x)
进来一个事件,acceptor进行接受,然后看看哪一个handler是对当前事件进行处理的,使用那个事件.
### nio的核心组件
```
channels 表示一个连接,无论是和文件还是和socket(TCP),都是支持非阻塞读的
```
```
Buffers 相当于是个数组,可以直接被channel读或者写的
```
然后就是核心组件
```
selector 分辨一系列有IO操作的Channel
```
```
selectorKey IO事件的状态和绑定
```
### 单线程Reactor实现
#### Reactor
##### part1
```
class Reactor implements Runnable
{
    final Selector selector;
    final ServerSocketChannel serverSocket;

    public Reactor(int port) throws Exception
    {
        selector=Selector.open();//开启选择器
        serverSocket=ServerSocketChannel.open();//开启一个服务端Channel
        serverSocket.bind(new InetSocketAddress(port));//绑定端口
        serverSocket.configureBlocking(false);//设置为非阻塞
        //serverSocket注册到选择器,然后监听连接的事件
        SelectionKey sk=serverSocket.register(selector, SelectionKey.OP_ACCEPT);
        //对于当前的SelectionKey,在绑定一个acceptor
        sk.attach(new Acceptor());
    }
```
Reactor的初始过程,就是一个选择器,一个服务器通道,当前的选择器选择连接这个事件.
##### part2
```
public void run() {
       try {
           //Reactor的循环  
           while(!Thread.interrupted())
           {     
               selector.select();
               //接受一系列的感兴趣的事件
               Set<SelectionKey> selectionKeys = selector.selectedKeys();
               //遍历这些感兴趣的事件
               Iterator<SelectionKey> it = selectionKeys.iterator();
               while(it.hasNext())
               {   //将不同的SelectorKey
                   //分发给不同的Handler();
                   dispatch((SelectionKey)it.next());
               }
           }
       }catch (Exception e)
       {

       }
    }
```
其中的分发Handler()
```
 void dispatch(SelectionKey k)
    {
        Runnable r=(Runnable)(k.attachment());
        if(r!=null)
        {
            r.run();
        }
    }
```
我们获取到每一个SelectorKey,然后如果是Accept的事件,k获取的就是acceptor,然后让这个线程运行
##### part3
在Reactor中的acceptor这个class是一个线程类
```
class Accpeptor implements Runnable
{

    @Override
    public void run() {
        try {
            SocketChannel c=serverSocket.accept();//建立连接
            if(c!=null)
            {
               new Handler(selector,c);//对于这个客户端的连接,我们要对它创建Handler
               //selector还是原来的那个selector,因为有了新的连接,所以我们要为这个selector注册更多的事件             //所以我们要将客户端连接和selector传入  
            }
        }catch (Exception e)
        {

        }

    }
}
```
##### part4
```
 final class Handler implements Runnable
 {
    final SocketChannel socket;
    final SelectionKey sk;
    ByteBuffer input=ByteBuffer.allocate(1024);
    ByteBuffer output=ByteBuffer.allocate(1024);
    static final int Reading = 0,Sending=1;
    int state=Reading;

    Handler(Selector sel,SocketChannel c) throws Exception
    {
        socket=c;
        sk=socket.register(sel,0);//Reading
        sk.attach(this);
        sk.interestOps(SelectionKey.OP_READ);
        sel.wakeup();
    }
    boolean inputIsComplete(){}
    boolean outputIsComplete(){}
    void process(){}
    @Override
    public void run() {
        try {
          if(state==Reading) read();
          else if(state==Sending) send();
        }catch (Exception e)
        {

        }
    }
    void read()
    {
       socket.write(output);
       if(outputIsComplete()) sk.cancel();
    }
    void send()
    {
       socket.read(input);
       if(inputIsComplete()) sk.cancel();
    }
}
```
![](_v_images/20190526161508537_1847745188.png =660x)
另一种handler的方式
![](_v_images/20190526161937654_1951193202.png =660x)
### 多线程Reactor实现
策略性的增加一些线程,适合多核处理器
对于上面的单线程处理器会减慢Reactor,将非IO的处理给其他线程完成(将Handler的线程交给其他的线程完成)
同时可以增加多个Reactor.
![](_v_images/20190526163232874_302833697.png =660x)
![](_v_images/20190526163757749_959124289.png =660x)
![](_v_images/20190526163924918_1480577241.png =660x)
![](_v_images/20190526163942689_862292307.png =660x)