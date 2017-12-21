#java启动脚本导入包问题

##背景
有一个项目使用启动脚本启动，发现Proxy.sh可以启动，而startup.sh启动失败。
``` log
    : command not found5: echo
    : No such file or directoryhome/rcsm/RCSP
    : command not found7: 
    .pid: No such file or directory
    '/startup.sh: line 11: syntax error near unexpected token `
    '/startup.sh: line 11: `if (test "$PROCESS_COUNT" = "1")
```

##步骤
1. 错误日志是找不到相关包，所以查看两个脚本引入包的情况。发现
``` shell
Prxoy.sh:
java -DjadoID=$APP_NAME -Dsmsc.log=$APP_HOME -Djava.ext.dirs=$APP_HOME/lib $CLASS_PATH
startup.sh:
java -DjadoID=$APP_NAME -Dsmsc.log=$APP_HOME -cp $CLASS_PATH
```
