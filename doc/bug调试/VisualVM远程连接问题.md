#VisualVM远程连接问题

##背景
为了能够远程监控服务器状态，以便进行调优，我使用了VisualVM工具。但是在使用前的配置过程中遇到了问题，以致本地无法获取到远程的服务器状态。故在此记录遇到的问题和解决的方案。

##环境
* server:CentOS 6.7, jdk 1.7
* client:Windows 8.1, jdk 1.8, VisualVM 1.3.9

##问题
按照教程中的步骤：
1. 创建文件jstatd.all.policy,
内容：grant codebase "file:${java.home}/../lib/tools.jar" {
    permission java.security.AllPermission;
};

2. 运行命令 jstatd -J-Djava.security.policy=jstatd.all.policy

因为在该步骤之后本地还是无法看到远程主机的情况，故增加了日志的输出，以便定位问题（-J-Djava.rmi.server.logCalls=true）。 
```
Mar 01, 2017 6:41:08 PM sun.rmi.server.UnicastServerRef logCall
FINER: RMI TCP Connection(6)-172.23.27.202: [172.23.27.202: sun.rmi.registry.RegistryImpl[0:0:0, 0]: void rebind(java.lang.String, java.rmi.Remote)]
Mar 01, 2017 6:41:16 PM sun.rmi.server.UnicastServerRef logCall
FINER: RMI TCP Connection(7)-172.23.11.56: [172.23.11.56: sun.rmi.registry.RegistryImpl[0:0:0, 0]: java.rmi.Remote lookup(java.lang.String)]
Mar 01, 2017 6:42:46 PM sun.rmi.server.UnicastServerRef logCall
FINER: RMI TCP Connection(8)-127.0.0.1: [127.0.0.1: sun.rmi.transport.DGCImpl[0:0:0, 2]: java.rmi.dgc.Lease dirty(java.rmi.server.ObjID[], long, java.rmi.dgc.Lease)]
```

##解决
查询该问题日志获知，产生原因有可能为服务端hostname映射的ip为127.0.0.1，导致client连接不了。而解决方案也是这个相关的：
1. jstatd -J-Djava.security.policy=jstatd.all.policy -J-Djava.rmi.server.hostname=172.23.27.201 &
2. 修改/etc/hosts文件，并用hostname命令修改hostname，再执行jstatd -J-Djava.security.policy=jstatd.all.policy即可。

ps:我原本打算对项目部分功能做耗时统计的，这个无法满足。
