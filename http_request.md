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
网络通信组件，能够用于执行IO操作
#### Selector
实现IO多路复用，通过Selector一个线程监听多个连接的Channel事件
#### NioEventLoop
维护一个线程和任务队列
#### NioEventLoopGroup
管理eventLoop的生命周期，类似线程池的概念
#### ChannelHandler
处理IO事件或拦截IO操作，并将其转发到ChannelPipeline中的下一个处理程序
#### ChannelHandlerContext
保存Channel的上下分
#### ChannelPipeline
持有ChannelHandler的list，控制ChannelHandler处理的出入栈操作
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

