# filter
```
Servlet过滤器的配置包括两部分：
第一部分是过滤器在Web应用中的定义，由<filter>元素表示，包括<filter-name>和<filter-class>两个必需的子元素
第二部分是过滤器映射的定义，由<filter-mapping>元素表示,可以将一个过滤器映射到一个或者多个Servlet或JSP文件，也可以采用url-pattern将过滤器映射到任意特征的URL。
```
配置过滤器自身的定义,然后在配置拦截的规则
![](_v_images/20190518103818073_1303025646.png =660x)
