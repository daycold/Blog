# redis
## sentinel
数据节点： 非sentinel节点

1. 每10s向主节点发送消息获取主从节点信息，并建立命令连接和订阅连接
2. 每2s通过master节点的channel广播当前sentinel节点的信息和对主节点的判断
3. 每秒ping一下其他节点

sentinel间的发现：

1. Sentinel 利用 pub/sub（发布/订阅）机制(如上2），订阅了每个 master 和 slave 数据节点的 __sentinel__:hello 频道，去自动发现其它也监控了统一 master 的 sentinel 节点
2. Sentinel 向每 1s 向 __sentinel__:hello 中发送一条消息，包含了其当前维护的最新的 master 配置。如果某个sentinel发现自己的配置版本低于接收到的配置版本，则会用新的配置更新自己的 master 配置
3. 与发现的 Sentinel 之间相互建立命令连接，之后会通过这个命令连接来交换对于 master 数据节点的看法

------

1. 当ping其他节点超时则主观判断该节点下线。
2. 当该节点为主节点时，sentinel会询问其他节点对主节点的判断，超过一定值认为该节点下线则客观下线。
3. 主观下线该节点的sentinel进行选举，选一台sentinel来执行下线节点的从节点升级主节点
4. 选择健康，优先级设置得高，偏移量大（保存数据多）的从节点升级为主节点
5. 新的主节点创建复制缓冲区，存放数据变更命令
