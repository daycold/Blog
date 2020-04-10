# HTTP_REQUEST
## NETTY
### 启动

    fun main () {
        ServerBootstrap().group(NioEventLoopGroup())
            .channel(NioServerSocket::class.java)
            .childHandler(object: ChannelInitializer<SocketChannel>() {
                override fun initChannel(ch:  SocketChannel) {}
            }).bind().sync()
    }
### 输出request的content
    class HttpServer(private val port: Int) {
        @Throws(Exception::class)
        fun start() {
            val bootstrap = ServerBootstrap()
            val group = NioEventLoopGroup()
            bootstrap.group(group).channel(NioServerSocketChannel::class.java)
                .childHandler(object : ChannelInitializer<SocketChannel>() {
                    override fun initChannel(ch: SocketChannel) {
                        ch.pipeline()
                            // 用于解码request
                            .addLast("decoder", HttpRequestDecoder())
                            // 用于编码response
                            .addLast("encoder", HttpResponseEncoder())
                            // 指定消息聚合器
                            .addLast("aggregator", HttpObjectAggregator(512 * 1024))
                            .addLast("handler", HttpHandler())
                    }
                })
                .option(ChannelOption.SO_BACKLOG, 128)
                .childOption(ChannelOption.SO_KEEPALIVE, true)
            bootstrap.bind(port).sync()
        }
    }

    class HttpHandler : SimpleChannelInboundHandler<FullHttpRequest>() {
        override fun channelRead0(ctx: ChannelHandlerContext, msg: FullHttpRequest) {
            // 请求body 
            val content = msg.content()
            val data = content.readBytes(content.readableBytes())
            val response = DefaultFullHttpResponse(
                HttpVersion.HTTP_1_1, HttpResponseStatus.OK,
                // 将请求body写入响应body中
                Unpooled.wrappedBuffer(data)
            )
            response.headers().apply {
                add(HttpHeaderNames.CONTENT_TYPE, "${HttpHeaderValues.TEXT_PLAIN}; charset=UTF-8")
                add(HttpHeaderNames.CONTENT_LENGTH, response.content().readableBytes())
                add(HttpHeaderNames.CONNECTION, HttpHeaderValues.KEEP_ALIVE)
        }
            ctx.write(response)
        }
    }
### 结构
#### Bootstrap
配置整个程序，引导启动类
#### ChannelFuture
所有IO操作都是异步
#### Channel
网络通信组件，能够用于执行IO操作（a channel represents an open connection to an entity)
#### Selector
实现IO多路复用，通过Selector一个线程监听多个连接的Channel事件
#### NioEventLoop
维护一个线程和任务队列（实现了ExecutorService接口），聚合一个Selector实现处理多个客户端连接
#### NioEventLoopGroup
管理eventLoop的生命周期，类似线程池的概念
#### ChannelHandler
处理IO事件或拦截IO操作，并将其转发到ChannelPipeline中的下一个处理程序
#### ChannelHandlerContext
保存Channel的上下分
#### ChannelPipeline
持有ChannelHandler的list，控制ChannelHandler处理的出入栈操作
###流程
NioEventLoop (里面的Selector)监听到注册的端口(channel)有输入进来，进行io操作，把输入放入任务列队，然后被线程池中的线程处理
## TOMCAT
### 启动
    
    fun main() {
        Tomcat().start()
    }
### 输出request的content
    fun main() {
        val tomcat = Tomcat()
        // 可以创建临时文件夹
        tomcat.setBaseDir("tomcat.8080")
        tomcat.host.autoDeploy = false
        val connector = Connector("HTTP/1.1").apply {
            port = 8080
        }
        tomcat.connector = connector
        val context = StandardContext().apply {
            name = "defaultContext"
            docBase = "default"
            addLifecycleListener(Tomcat.FixContextListener())
            // 指定该context处理的请求路径，"" 为默认，找不到处理器时给它处理
            path = ""
        }
        tomcat.host.addChild(context)
        tomcat.addServlet("defaultContext", "defaultServlet", TomcatServlet())
        // 将context处理的指定路径分配给指定servlet处理
        context.addServletMappingDecoded("/test", "defaultServlet")
        tomcat.engine
        tomcat.start()
    }

    class TomcatServlet : Servlet {
        private var servletConfig: ServletConfig? = null

        override fun getServletConfig(): ServletConfig? {
            return servletConfig
        }

        override fun destroy() {
        }

        override fun init(config: ServletConfig?) {
            this.servletConfig = config
        }

        override fun getServletInfo(): String {
            return "default servlet"
        }

        override fun service(req: ServletRequest, res: ServletResponse) {
            req.inputStream.use { stream ->
                res.outputStream.use { out ->
                    stream.transferTo(out)
                }
            }
        }
    }
    
### 结构
[参考](https://www.cnblogs.com/kismetv/p/7228274.html)
#### Server
最顶层，代表整个tomcat容器
port表示server接收shutdown的接口
主要任务，提供一个接口让客户端能够访问到这个Service集合
#### Service
不同Service监听不同接口
一个Service可以有多个Connector，但是只有一个Engine
Connector接收请求，Engine处理请求
####Connector
接受连接请求，创建Request和Response对象，并交给Engine处理
#### Engine
请求处理组件，可以内嵌一个或多个Host
#### Host
代表Engine的一个虚拟主机，至少有一个的name与Engine的defaultHost匹配
name指定了虚拟主机名，如localhost，127.0.0.1, www.baidu.com
#### Context
代表特定虚拟主机上运行的一个Web应用
docBase指定了Web应用使用的WAR包路径，自动部署下，不在appBase目录中才需要指定
path指定访问该web应用的上下文路径，根据path与相应uri的匹配程度来选择相应应用处理
自动部署下不能指定path属性，将推导出来
context可以添加多个servlet来处理分配给它的更详细路径的请求
#### Listener
可以监听Server,Engine,Host,Context
## UNDERTOW
### 启动
    fun main() {
        Undertow.builder().build().start()
    }
### 输出request的content
    fun main() {
        Undertow.builder().addHttpListener(8080, "127.0.0.1")
        // 普通的HttpHandler拿不到inputStream
        .setHandler(BlockingHandler(UndertowHandler())).build().start()
    }

    class UndertowHandler : HttpHandler {
        override fun handleRequest(exchange: HttpServerExchange) {
            exchange.statusCode = 200
            exchange.inputStream.use { stream ->
                exchange.outputStream.use { out ->
                    stream.transferTo(out)
                }
            }
        }
    }
### 结构
#### HttpHandler
处理HttpServerExchange
##### PathHandler
处理路由
##### BlockingHandler
调用了exchange.startBlocking()方法，能够获取到连接中的inputStream

exchange中的io流通过blockingHttpExchange对象获得。调用该方法会生成一个新的blockingHttpExchange对象，可以得到io流
#### HttpServerExchange
持有请求和响应的上下文信息
#### ListenerConfig
指定协议端口号等配置

## JETTY
### 启动
    fun main() {
        Server().start()
    }
### 输出request的content
    fun main() {
        val server = Server(8080)
        server.handler = JettyHandler(server)
        server.start()
    }

    // servlet包中有 ServletContextHandler 的实现，可以直接添加 servlet
    class JettyHandler(server: Server) : Handler by server {
        override fun handle(
            target: String?,
            baseRequest: Request,
            request: HttpServletRequest,
            response: HttpServletResponse
        ) {
            request.inputStream.use { stream ->
                response.outputStream.use { out ->
                    stream.transferTo(out)
                }
            }
        }
    }
### 结构
#### Handler
处理请求并响应
Server类本身继承Handler
Handler继承LifeCycle接口实现生命周期管理
#### Connector
直接接受客户端连接的抽象，一个Connector监听Jetty服务器的一个端口
如调用Server(8080)时，创建了 Connector(8080)并添加到Server持有的连接队列中
继承LifeCycle实现生命周期管理
继承Container实现容器功能

## Http
请求协议示例

GET /api/v2/infos HTTP/1.1
Content-Type: application/json
User-Agent: PostmanRuntime/7.22.0
Accept: */*
Cache-Control: no-cache
Postman-Token: d04ae105-918c-4776-bd24-d8c2f3204e72
Host: student-api.beta.saybot.net:80
Accept-Encoding: gzip, deflate, br
Cookie: JSESSIONID=j5CU4JGHJ622Kd-oCifgGHasbnQ2gxLPUgedbvH6
Connection: keep-alive


(后面需一行空行。空行后再放body（如果有的话，没有就再空一行），有body需要加content-length的请求头）

实质只是socket发送一段固定格式的文本

## Springboot-web-mvc
Q:
spring做了什么
A:
spring申明了WebServer的模型，申明了webServerFactory 来创建 WebServer， 并在 springContext 的特定生命周期中启动 WebServer
Q:
web容器怎么将请求交给spring处理
A:
创建 webServer 的时候会将 servlet 容器传入 webServerFactory，在这个过程中实现了 servlet 与 web 容器的关联。

### springweb 申明的结构
#### WebServer 接口
springWeb中申明WebServer接口，申明 start, stop, getPort方法
在 springContext 的 onRefresh 的时候创建，finishRefresh 阶段会调用start方法

#### AbstractServletWebServerFactory 抽象类
提供一些对session，ssl等的支持
实现了 WebServerFactory 接口
#### ServletWebServerFactory 接口
申明 getWebServer 方法，支持传入过多ServletContext 对象，ServletContext 对象为 Servlet 容器

#### AbstractReactiveWebServerFactory 抽象类
响应式的web容器工厂

#### WebServerFactory 接口
空接口，标记为webServerFactory

#### WebServerFactoryCustomizer 接口
对 webServerFactory 添加定制信息，在ServletWebFactoryAutoConfiguration注入的时候注入

#### DispatcherServlet 类
支持路径分发功能，在webServerFactoryCustomizer 注入后注入，springweb 默认生成的servlet类。undertow，tomcat接入spring都是对其的包装或直接使用。

### coroutine来当web容器的调度器
#### Undertow
DeploymentInfo 中添加 initialHandlerChainWrapper，使用协程对 Handler 进行包装

#### Tomcat
在 TomcatServletWebServerFactory 中添加 TomcatContextCustomizer，指定支持协程的 wrapper。wrapper中包装 servlet，开启异步，使用协程。

#### 问题
controller怎么织入协程。
suspend 方法应该要直接放在协程作用域中。因此，需要在调用 controller 方法的地方织入


