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