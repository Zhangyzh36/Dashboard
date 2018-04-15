# [关于Web服务器、Web框架和WSGI的理解](https://controny.github.io/2018/04/15/web-server-web-app-and-wsgi/)

项目确定了用Flask作为服务端开发框架，但阅读了其他项目的文档，发现他们对于后台的架构设计还总是提到Nginx、WSGI，一时间有些迷茫，查阅了一些资料后，才理清了这三者的关系。

总的来说，客户端从发送一个HTTP请求到Flask处理请求，分别经过了Web服务器层，WSGI层，Web框架层这三个层次。
![](https://controny.github.io/assets/images/posts/20180415094504.png)
下面就分别简单介绍一下这三个层次。

## Web服务器
当我们在浏览器输入URL后，浏览器会先请求DNS服务器，获得请求站点的 IP 地址。然后发送一个HTTP Request（请求）给拥有该 IP 的主机，接着就会接收到服务器给我们的HTTP Response（响应），浏览器经过渲染后，以一种较好的效果呈现给我们。这个过程中，正是Web服务器在幕后默默做贡献。
![](https://controny.github.io/assets/images/posts/20180415094741.png)
Web服务器的基本工作原理并不复杂，一般可分成如下4个步骤：建立连接、请求过程、应答过程以及关闭连接。但在实际使用中，Web服务器的角色并不简单，除了简单发送静态文件给客户端外，还要考虑缓存机制、安全、并发处理、记录日志等。

上述的Nginx便是Web服务器的代表。

## Web框架
Web服务器接受Http Request，返回Response，很多时候Response并不是静态文件，因此需要有一个**Web应用程序**根据Request生成相应的Response。这里的应用程序主要用来处理相关业务逻辑，读取或者更新数据库，根据不同Request返回相应的Response。注意这里并不是Web服务器本身来做这件事，它只负责Http协议层面和一些诸如并发处理、安全、日志等相关的事情。
Web应用程序可以用各种语言编写（Java, PHP, Python, Ruby等），它会从Web服务器接收客户端的请求，处理完成后，再返回响应给Web服务器，最后由Web服务器返回给客户端。
![](https://controny.github.io/assets/images/posts/20180415095637.png)
早期开发Web应用程序做了许多重复性劳动，后来为了减少重复，避免写出庞杂，混乱的代码，人们将Web应用程序的关键性过程提取出来，开发出了各种**Web框架**。有了框架，就可以专注于编写清晰、易维护的代码，无需关心数据库连接之类的重复性工作。

Flask就是这样一种Web框架。

## WSGI
Python有着许多的Web框架，而同时又有着许多的Web 服务器（除了Nginx以外，还有Apache等），Web框架和Web服务器之间需要进行通信，如果在设计时它们之间不可以相互匹配的，那么选择了一个框架就会限制对Web服务器的选择，这显然是不合理的。

怎样确保可以在不修改Web服务器代码或Web框架代码的前提下，使用自己选择的服务器，并且匹配多个不同的网络框架呢？答案是接口，设计一套双方都遵守的接口就可以了。对Python来说，就是**WSGI**（Web Server Gateway Interface，Web服务器网关接口）。其他编程语言也拥有类似的接口：例如Java的Servlet API和Ruby的Rack。

Python WSGI的出现，让开发者可以将Web框架与Web服务器的选择分隔开来，不再相互限制。现在，你可以真正地将不同的Web服务器与Web框架进行混合搭配，选择满足自己需求的组合。
![](https://controny.github.io/assets/images/posts/20180415100334.png)

## 小结
至此，我对项目后台的架构设计有了一定的心得:

1. 以Nginx作为项目的Web服务器，提供静态文件服务。
2. 以Flask作为Web框架，处理业务逻辑。
3. Flask通过WSGI与Nginx交互。

