# cookie
## 1
1.session用来表示用户会话，session对象在服务端维护，一般tomcat设定session生命周期为30分钟，超时将失效，也可以主动设置无效
2.cookie存放在客户端，可以分为内存cookie和磁盘cookie。内存cookie在浏览器关闭后消失，磁盘cookie超时后消失。当浏览器发送请求时，将自动发送对应cookie信息，前提是请求url满足cookie路径；
3.可以将sessionId存放在cookie中，也可以通过重写url将sessionId拼接在url。因此可以查看浏览器cookie或地址栏url看到sessionId； 
4.请求到服务端时，将根据请求中的sessionId查找session，如果可以获取到则返回，否则返回null或者返回新构建的session，老的session依旧存在.

```
有关会话跟踪技术描述正确的是（）
正确答案: A B C 
Cookie是Web服务器发送给客户端的一小段信息，客户端请求时，可以读取该信息发送到服务器端
关闭浏览器意味着临时会话ID丢失，但所有与原会话关联的会话数据仍保留在服务器上，直至会话过期
在禁用Cookie时可以使用URL重写技术跟踪会话
隐藏表单域将字段添加到HTML表单并在客户端浏览器中显示
```