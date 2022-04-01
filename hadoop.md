# HADOOP
## hdfs
Hadoop Distributed File System，分布式文件系统

一次写入多次读取的文件访问模型

主要包括三个角色：client, NameNode和DataNode

### client
通过api访问数据
### nameNode
管理系统命名空间；管理 block 所在 dataNode
### dataNode
存储节点，dataNode 为集群，数据被分割为 block，一个完整的文件分割成多个 block 分布在集群的多个 dataNode 中。每个 block 可以根据配置在别的 dataNode 中存在备份。