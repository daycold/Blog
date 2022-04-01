#LINUX
## Zookeeper
### mac commands
zkServer start, stop... (start or stop default cfg file)
zkServer start xxxx.cfg
zkCli
### centos
将文件解压到一个目录,如 /usr/local/services
修改 /etc/profile 文件
    
    export ZOOKEEPER_HOME=/usr/local/services/{解压目录}
    export PATH=$PATTH:$ZOOKEEPER_HOME/bin:$PATH:ZOOKEEPER_HOME/conf

    zkServer.sh start
## Kafka
  需要设置advertised.listeners
### commands
kafka-server-start /usr/local/etc/kafka/server.properties
## Common
ps (short for Process Status)
echo 追加文本
ls -l 查看文件权限
ls -ld 查看目录权限

    drwxr-xr-x root root 4096 timestamp file/dir
    第一位文件类型 d: Dir, l: link, -: common file, p: pipe(管道)
    2-4位，文件属主权限， r读,w写,x执行
    5-7位，文件属主同组用户的权限
    8-10位，其他用户的权限
    文件拥有者
    文件拥有者所在组
    文件大小（字节）
    时间日期
    文件名

chown [-R(递归)] {user}:{group} {file/dir} 修改文件所有者

jps java process status? 查看java程序状态？

## shell
chsh 切换shell(zsh, bash)等

~/.bashrc 中添加别名

    如：
      alias='/usr/local/services/kafka/bin/zkServer.sh'

      就可以将 /usr/local/servics/kafka/bin/zkServer.sh start 替换为 zkServer start

/etc/profile 中添加环境变量
    
    如
      export KAFKA_HOME='/usr/local/services/kafka'
      export PATH=$PATH:$KAFKA_HOME/bin:$KAFKA_HOME/conf
      就可以 zkServer.sh start zoo.cfg #zkServer.sh 在/usr/local/services/kafka/bin 目录下，zoo.cfg在其conf目录下
### plugin
#### autojump
j + 关键字 自动跳转目录

## bash

## ssh
    useradd {username} # 创建用户 -g 参数指定组，否则会新建用户名相同的组
	passwd {username} # 设置密码
	usermod -g {group} {username} # 变更用户主要组，加-a参数则只添加组
	groupadd {group}
	groupdel {group}
	groupmod {group}
	gpasswd -d {username} {group} # 组中移除用户
    ssh-keygen -t rsa -C "email" # 生成ssh

scp local_file remote_username@remote_ip:remote_folder
	
配置文件在 /etc/ssh/sshd_config, service sshd restart 重启生效配置

1. 添加 AuthorizedKeysFile      .ssh/authorized_keys 和 PubkeyAuthentication yes 允许ssh免密登陆
2. AllowUsers 和 DenyUsers 配置黑白名单

## POSTGRESQL
第一次登陆的时候一定要切换到 postgres 用户，并且添加sudo权限启动项目。然后可以去添加自己的role。

