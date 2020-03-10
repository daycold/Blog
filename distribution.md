# DISTRIBUTION
## Docker
### notice
1. 在使用docker image ls等命令前必须先启动docker（systemtrl start docker)
## ZooKeeper
## Nginx
### structure
一个master，多个client，由master集中管理。每个client占用一个进程，配置项中最大请求数为单个client的最大请求数。采用nio，支持pipe在一个连接中发起多个请求。在反代服务中，由于同时需要与客户端和服务器通信，实际最大请求数要减半。
