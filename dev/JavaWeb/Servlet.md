## Servlet生命周期
```
Servlet 加载—>实例化—>服务—>销毁。

1. init（）：
在Servlet的生命周期中，仅执行一次init()方法。
它是在服务器装入Servlet时执行的，负责初始化Servlet对象。
可以配置服务器，以在启动服务器或客户机首次访问Servlet时装入Servlet。
无论有多少客户机访问Servlet，都不会重复执行init（）。
2. service（）：
它是Servlet的核心，负责响应客户的请求。
每当一个客户请求一个HttpServlet对象，该对象的Service()方法就要调用，
而且传递给这个方法一个“请求”（ServletRequest）对象和一个“响应”（ServletResponse）对象作为参数。
在HttpServlet中已存在Service()方法。
默认的服务功能是调用与HTTP请求的方法相应的do功能。
3. destroy（）：
仅执行一次，在服务器端停止且卸载Servlet时执行该方法。
当Servlet对象退出生命周期时，负责释放占用的资源。
一个Servlet在运行service()方法时可能会产生其他的线程，
因此需要确认在调用destroy()方法时，这些线程已经终止或完成。
```
## Tomcat 与 Servlet 是如何工作的

![](24114945-4774512d1247438fa58c37399d3999ae.jpg)
步骤：

* Web Client 向Servlet容器（Tomcat）发出Http请求
* Servlet容器接收Web Client的请求
* Servlet容器创建一个HttpRequest对象，将Web Client请求的信息封装到这个对象中。
* Servlet容器创建一个HttpResponse对象
* Servlet容器调用HttpServlet对象的service方法，把HttpRequest对象与HttpResponse对象作为参数传给HttpServlet 对象。
* HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息。
* HttpServlet调用HttpResponse对象的有关方法，生成响应数据。
* Servlet容器把HttpServlet的响应结果传给Web Client。
  
## Servlet工作原理

  1. 首先简单解释一下Servlet接收和响应客户请求的过程，首先客户发送一个请求，Servlet是调用service()方法对请求进行响应的，通过源代码可见，service()方法中对请求的方式进行了匹配，选择调用doGet,doPost等这些方法，然后再进入对应的方法中调用逻辑层的方法，实现对客户的响应。在Servlet接口和GenericServlet中是没有doGet（）、doPost（）等等这些方法的，HttpServlet中定义了这些方法，但是都是返回error信息，所以，我们每次定义一个Servlet的时候，都必须实现doGet或doPost等这些方法。

  2. 每一个自定义的Servlet都必须实现Servlet的接口，Servlet接口中定义了五个方法，其中比较重要的三个方法涉及到Servlet的生命周期，分别是上文提到的init(),service(),destroy()方法。GenericServlet是一个通用的，不特定于任何协议的Servlet,它实现了Servlet接口。而HttpServlet继承于GenericServlet，因此HttpServlet也实现了Servlet接口。所以我们定义Servlet的时候只需要继承HttpServlet即可。

  3. Servlet接口和GenericServlet是不特定于任何协议的，而HttpServlet是特定于HTTP协议的类，所以HttpServlet中实现了service()方法，并将请求ServletRequest、ServletResponse强转为HttpRequest 和 HttpResponse。

### 创建Servlet对象的时机：

* Servlet容器启动时：读取web.xml配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，同时将ServletConfig对象作为参数来调用Servlet对象的init方法。
* 在Servlet容器启动后：客户首次向Servlet发出请求，Servlet容器会判断内存中是否存在指定的Servlet对象，如果没有则创建它，然后根据客户的请求创建HttpRequest、HttpResponse对象，从而调用Servlet对象的service方法。
* Servlet：
Servlet容器在启动时自动创建Servlet，这是由在web.xml文件中为Servlet设置的属性决定的。从中我们也能看到同一个类型的Servlet对象在Servlet容器中以单例的形式存在。

## Get和Post方法的区别
1. GET用于从指定资源请求数据。查询字符串（名称/值对）是在GET请求的URL中发送的。
   ```
    * GET请求可以被缓存
    * GET请求保留在浏览器历史记录中
    * GET请求可以加书签
    * 处理敏感数据时，切勿使用GET请求
    * GET请求有长度限制
    * GET请求仅用于请求数据（不修改）
   ```
2. POST用于将数据发送到服务器以创建/更新资源。
通过POST发送到服务器的数据存储在HTTP请求的请求主体中.
   ```
    * POST请求永远不会被缓存
    * POST请求不会保留在浏览器历史记录中
    * POST请求无法添加书签
    * POST请求对数据长度没有限制
   ```
3. PUT用于将数据发送到服务器以创建/更新资源。
   POST和PUT之间的区别在于PUT请求是幂等的。也就是说，多次调用相同的PUT请求将始终产生相同的结果。相反，重复调用POST请求会产生多次创建相同资源的副作用。
4. HEAD与GET几乎相同，但是没有响应主体。
换句话说，如果GET / users返回用户列表，那么HEAD / users将发出相同的请求，但不会返回用户列表。
HEAD请求对于在实际发出GET请求之前（例如在下载大文件或响应正文之前）检查GET请求返回的内容很有用。
5. DELETE方法删除指定的资源。
6. OPTIONS方法描述了目标资源的通信选项。
7. 