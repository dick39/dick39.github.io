#tomcat关闭异常

##背景
因tomcat7关闭异常
```
Dec 20, 2016 9:36:08 AM org.apache.catalina.core.StandardServer await
INFO: A valid shutdown command was received via the shutdown port. Stopping the Server instance.
Dec 20, 2016 9:36:08 AM org.apache.coyote.AbstractProtocol pause
INFO: Pausing ProtocolHandler ["http-nio-8100"]
Dec 20, 2016 9:36:08 AM org.apache.catalina.core.StandardService stopInternal
INFO: Stopping service Catalina
[410174158 INFO  2016/12/20 09:36:08:281][org.springframework.context.support.AbstractApplicationContext] Closing Root WebApplicationContext: startup date [Thu Dec 15 15:39:54 CST 2016]; root of context hierarchy
[410174158 DEBUG 2016/12/20 09:36:08:281][org.springframework.beans.factory.support.AbstractBeanFactory] Returning cached instance of singleton bean 'sqlSessionFactory'
[410174158 DEBUG 2016/12/20 09:36:08:281][org.springframework.beans.factory.support.AbstractBeanFactory] Returning cached instance of singleton bean 'lifecycleProcessor'
[410174158 INFO  2016/12/20 09:36:08:281][org.springframework.beans.factory.support.DefaultSingletonBeanRegistry] Destroying singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@67995af1: defining beans [*bean*]; root of factory hierarchy
Dec 20, 2016 9:36:08 AM org.apache.catalina.loader.WebappClassLoader clearReferencesJdbc
SEVERE: The web application [/ai_httpproxy] registered the JDBC driver [com.mysql.jdbc.Driver] but failed to unregister it when the web application was stopped. To prevent a memory leak, the JDBC Driver has been forcibly unregistered.
Dec 20, 2016 9:36:08 AM org.apache.catalina.loader.WebappClassLoader clearReferencesThreads
SEVERE: The web application [/ai_httpproxy] appears to have started a thread named [NioProcessor-2] but has failed to stop it. This is very likely to create a memory leak.
Dec 20, 2016 9:36:08 AM org.apache.catalina.loader.WebappClassLoader clearReferencesThreads
SEVERE: The web application [/ai_httpproxy] appears to have started a thread named [NioProcessor-17] but has failed to stop it. This is very likely to create a memory leak.
Dec 20, 2016 9:36:08 AM org.apache.coyote.AbstractProtocol stop
INFO: Stopping ProtocolHandler ["http-nio-8100"]
Dec 20, 2016 9:36:08 AM org.apache.coyote.AbstractProtocol destroy
INFO: Destroying ProtocolHandler ["http-nio-8100"]
```
查询了资料和大牛的分析后，自己进行了整理，开始想办法解决这个问题。

##原因
tomcat收到stop命令后，未注销掉的JDBC Driver抛出告警。

#解决方案
1. server.xml
在tomcat6中加入了org.apache.catalina.core.JreMemoryLeakPreventionListener内存泄露监听。不注册该监听器就看不到告警。但是这个并没有解决该问题，只在此做记录。
2. jdbc Driver包位置
将mysql-connector-java-{version}.jar放入tomcat/lib目录下。经我在tomcat7中测试，关闭tomcat不再有该告警。但是这样是否会影响我们部署tomcat的便捷性，需要记住将该jar包放入tomat的lib，而不是项目的lib。
3. 重写注销是时反注册的代码
这一块找到写的最好的是[tomcat-jdbc-leak-prevention](http://hongjiang.info/tomcat-jdbc-leak-prevention/)。
主要就是在WebappClassLoader调用clearReferencesJdbc时，我们重写相关的DataSource类:
```java
public class DataSource extends org.apache.commons.dbcp.BasicDataSource{

    @Override
    public synchronized void close() throws SQLException {
        DriverManager.deregisterDriver(DriverManager.getDrivers().nextElement());
        super.close();
    }
}
```
但是在我debug断点测试的几次里，均没有进入该方法，故还有当前问题。

#最后
我并没有完美解决该问题，在dbcp项目中也有人提出该[bug](https://issues.apache.org/jira/browse/DBCP-332)。现在的办法验证成功的只有方案2。
话说很想用一下阿里的druid，不知道会不会有变化。