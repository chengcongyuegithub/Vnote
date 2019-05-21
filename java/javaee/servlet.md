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
Servlet生命周期分成3个阶段：

1）初始化阶段：**调用init方法**

2）响应客户请求：**调用service**

3）终止：**调用destory方法**

 

初始化阶段：在下列时刻servlet容器装载servlet

1  servlet容器启动时，自动装载某些servlet

2  在servlet容器启动后，客户首次向servlet发送请求

3  servlet类文件被更新之后，重新装载servlet



Servlet被装载之后，servlet容器创建一个servlet'对象并调用servlet的init方法，在servlet生命周期内，init方法只能被调用一次。servlet工作原理：客户端发起一个请求，servlet调用service方法时请求进行响应，service对请求的方式进行了匹配，选择调用dopost或者doget等这些方法，然后进入对应方法中调用逻辑层的方法，实现对客户的响应。



响应客户请求：对于用户到达servlet的请求，servlet容器会创建特定于该请求的servletrequest和servletresponse对象，然后调用servlet的service方法，service方法从servletrequest对象中获取客户请求的信息，处理该请求，并且通过servletresponse对象向客户端返回响应信息。

 

终止：当web应用终止或者servlet容器终止或servlet容器重新装载servlet新实例时，servlet容器会调用servlet对象的destory方法，在destory方法中可以释放servlet占用的资源


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
## HttpServletRequest类

1.读取和写入HTTP头标

2.取得和设置cookies

3.取得路径信息

4.标识HTTP会话

## 题1
```
下面有关servlet service描述错误的是？
正确答案: B   你的答案: B (正确)
不管是post还是get方法提交过来的连接，都会在service中处理
doGet/doPost 则是在 javax.servlet.GenericServlet 中实现的
service()是在javax.servlet.Servlet接口中定义的
service判断请求类型，决定是调用doGet还是doPost方法
```
GenericServlet 抽象类 给出了设计 servlet 的一些骨架，定义了 servlet 生命周期，还有一些得到名字、配置、初始化参数的方法，其设计的是和应用层协议无关的

## 题2
```
如何获取ServletContext设置的参数值？
正确答案: B   你的答案: C (错误)
context.getParameter()
context.getInitParameter()
context.getAttribute()
context.getRequestDispatcher()
```
getParameter()是获取POST/GET传递的参数值；
getInitParameter获取Tomcat的server.xml中设置Context的初始化参数
getAttribute()是获取对象容器中的数据值；
getRequestDispatcher是请求转发。 
```
Web容器在启动时为每个Web应用创建一个ServletContext对象(**一个项目对应这一个ServletContext对象**)，ServletConfig对象中维护了ServletContext的引用，开发人员在编写servlet时，可以通过ServletConfig.getServletContext方法获得ServletContext对象。由于一个WEB应用中的所有Servlet共享同一个ServletContext对象，因此Servlet对象之间可以通过ServletContext对象来实现通讯。ServletContext对象通常也被称之为context域对象。
```
```
多个Servlet通过ServletContext对象实现数据共享。 在InitServlet的Service方法中利用ServletContext对象存入需要共享的数据 ServletContext context = this.getServletContext();    
context.setAttribute("name", "haha"); 

```
```
在其它的Servlet中利用ServletContext对象获取共享的数据   

ServletContext context = this.getServletContext();    String name = context.getAttribute("name");   

2. 获取WEB应用的初始化参数。 在DemoServlet的doPost方法中测试获取初始化参数的步骤如下:   

ServletContext context = this.getServletContext();   
String url = context.getInitParameter("url"); 
```