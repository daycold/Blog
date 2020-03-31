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
