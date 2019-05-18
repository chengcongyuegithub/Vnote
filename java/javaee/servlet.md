# servlet
## servlet生命周期
```
Servlet的生命周期
1.加载：容器通过类加载器使用Servlet类对应的文件来加载Servlet
2.创建：通过调用Servlet的构造函数来创建一个Servlet实例
3.初始化：通过调用Servlet的init()方法来完成初始化工作，这个方法是在Servlet已经被创建，但在向客户端提供服务之前调用。
4.处理客户请求：Servlet创建后就可以处理请求，当有新的客户端请求时，Web容器都会创建一个新的线程来处理该请求。接着调用Servlet的
Service()方法来响应客户端请求（Service方法会根据请求的method属性来调用doGet（）和doPost（））
5.卸载：容器在卸载Servlet之前需要调用destroy()方法，让Servlet释放其占用的资源。
```
常见错误
```
在创建自己的Servlet时候，应该在初始化方法init()方法中创建Servlet实例(X)
destroy()方法仅执行一次，即在服务器停止且卸载Servlet时执行该方法(true)
```

## servlet类的结构
![](_v_images/20190518104017891_940703989.png =660x)
```
下面有关servlet的层级结构和常用的类，说法正确的有?
A,GenericServlet类：抽象类，定义一个通用的、独立于底层协议的Servlet。
B,大多数Servlet通过从GenericServlet或HttpServlet类进行扩展来实现
C,ServletConfig接口定义了在Servlet初始化的过程中由Servlet容器传递给Servlet得配置信息对象
D,HttpServletRequest接口扩展ServletRequest接口，为HTTP Servlet提供HTTP请求信息
```
HttpServlet是GenericServlet的子类。
GenericServlet是个抽象类，必须给出子类才能实例化。它给出了设计servlet的一些骨架，定义了servlet生命周期，还有一些得到名字、配置、初始化参数的方法，其设计的是和应用层协议无关的，也就是说 你有可能用非http协议实现它。
HttpServlet是子类，当然就具有GenericServlet的一切特性，还添加了doGet, doPost, doDelete, doPut, doTrace等方法对应处理http协议里的命令的请求响应过程。
一般没有特殊需要，自己写的Servlet都扩展HttpServlet 。