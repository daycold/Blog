# DISTRIBUTION
## Docker
### notice
1. 在使用docker image ls等命令前必须先启动docker（systemtrl start docker)
2. 删除image前需要关掉container（docker ps -as)
### command
1. docker image 
2. docker container
3. docker build (-t name) .(上下文目录)
### dockfile
1. RUN 每条都会生成一个新的镜像层，可以使用&&合并多条指令
2. ADD 添加文件到镜像
3. FROM 镜像源文件
4. ENTRYPOINT 启动方法，如 ENTRYPOINT ['java', '-jar', 'xxx.jar']
5. EXPORT 暴露端口号
## ZooKeeper
## Nginx
### structure
一个master，多个client，由master集中管理。每个client占用一个进程，配置项中最大请求数为单个client的最大请求数。采用nio，支持pipe在一个连接中发起多个请求。在反代服务中，由于同时需要与客户端和服务器通信，实际最大请求数要减半。
### nginx.conf
1. user {用户名} {群组}; *为当前机器的属性，与文件权限相关，群组缺省时默认为与用户名称相同*
2. server *虚拟主机配置*
3. include {filename}; *引用文件，可以在各个域中使用*
#### http
1. upstream {名称} *申明服务器集群并命名*
2. server *配置具体服务或集群*
##### upstream
1. server {域名或ip + 端口}; *该服务名对应的服务器（可以有多个，列多个server)*
###### server
1. listen {端口号}; 
2. server_name {服务名};*通过匹配服务名决定将请求交给哪个server处理(该服务名需指向该机器)*
3. location {路径} *分配指定路径到指定服务器，全部为 '/'* 
4. root {路径}; *静态文件路径*
5. access_log {路径}; *访问记录日志*
6. error_log {路径}; *异常日志*
###### location

