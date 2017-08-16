#tomcat中war包部署的细节点

##背景
tomcat常用部署方式分为三种：
1. webapps下准备好文件夹
2. webapps下放入war包
3. conf/server.xml中进行设置``` <Context docBase="/usr/local/apps/test.war" path="/test" reloadable="true"/> ```

##启动使用优先级
经过尝试，在tomcat7环境下，优先级为上述编号的1>3>2。同一项目名再优先级高的条件符合时，不会再向下尝试。

##热部署
tomcat6开始，支持热部署，版本关键词为“ **##** ”。
例如首先将test##1.war放入webapps/目录并启动，访问已有接口localhost:8080/test/testHotDeploy，返回helloworld;
接着需求修改，需要将输出helloworld改为HelloWorld，可以将项目编译后打包，命名为test##2.war放入webapps/目录；
再次请求localhost:8080/test/testHotDeploy，返回helloworld；打开新浏览器访问该链接会返回HelloWorld。