# SPRING_CLOUD
## 依赖
spring-cloud-starter-netflix-eureka-client
spring-cloud-starter-netflix-eureka-server

整合了ribbon等。

##关系
client向server注册
当通过 http:#{APPLICATION-NAME}/path 发起通信时，client向server获取到目标application的信息，然后自己实行负载均衡

## 注解
name | description
--- | ---
EnableEurekaServer | 标记为server
EnableEurekaClient | 标记为client(该注解仅为标记性注解，是否启用功能配置 eureka.client.enable=true/false
EnableDiscoveryClient | 标记为client并能向server请求获取其他client的信息
LoadBalanced | 标记一个restTemplate使用特定的api实现负载均衡
EnableFeignClients | 标记为Feign服务
FeignClient | 使用rpc的方式访问特定api实现负载均衡

## 名词
name | description
--- | ---
雪崩 | 单个服务出现问题，导致依赖的服务全部阻塞，依赖的服务请求量大的时候也会耗尽servlet线程，传递到下一个依赖它的服务
断路器 | 特定服务不可用达到一定阈值，断路器会打开并返回固定值避免连锁故障

## 技术框架
name |description 
--- | ---
eureka / ˌjʊ(ə)ˈriːkə/| 服务发现组件，注册管理各个服务
ribbon /ˈrɪb(ə)n/| 使用restTemplate做负载均衡
hystrix /hɪst'rɪks/| 断路器
feign /fein/| 使用注解rpc做负载均衡，自带断路器
zuul | 网管配置
springCloudConfig | 分布式配置中心(指定远程配置文件地址，文件名为{服务名}-{env}.properties)
bus-amqp | 使用消息队列来广播配置文件更改或者服务之间通讯
sleuth /sluːθ/(zipkin) | 服务链路追踪

## 负载均衡
配置方式
{服务名}.ribbon.NFLoadBalancerRuleClassName=
配置类型

1. com.netflix.loadbalancer.RandomRule #配置规则 随机
2. com.netflix.loadbalancer.RoundRobinRule #配置规则 轮询
3. com.netflix.loadbalancer.RetryRule #配置规则 重试
4. com.netflix.loadbalancer.WeightedResponseTimeRule #配置规则 响应时间权重
5. com.netflix.loadbalancer.BestAvailableRule #配置规则 最空闲连接策略

## feign
可以通过覆盖Feign.Builder的InvocationHandlerFactory可以实现日志插入
