#Spark简单体会

##背景
因个人兴趣学了scala，而项目刚好要考察spark大数据分析是否适合我们项目，开始进行spark学习。

##SimpleApp
####自定义需求
对项目的前端点击请求进行统计，看看运营方使用最多的功能模块。

####实现想法
在学会demo的基础上，基本已无难点
```scala
    val logFileStr = "/home/sheng/Documents/cmslog/localhost_access_log.*"
    val conf = new SparkConf().setAppName("LogAnalysis").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val logFile = sc.textFile(logFileStr)
    //先排除静态元素，再对每行请求进行split取需要信息，最后合并计数排序
    val lineMap = logFile.filter((str)=>{!(str.toString.contains("jcaptcha.jpg") || str.toString.contains("/js/")
      || str.toString.contains("/shared_") || str.toString.contains("/images/") || str.toString.contains("/css/")
      || str.toString.contains("/fonts/"))})
      .map(line => (line.split(" "){6}.split("\\?"){0},1)).reduceByKey((x,y)=>{x+y}).sortBy(value => value._2,false)
    lineMap.saveAsTextFile("/home/sheng/Documents/output/cmslog")
```
唯有需要注意sc.textFile方法可以读取文件夹下匹配的文件。

####进一步调整
1. reduceByKey((x,y)=>{x+y})也可以使用foldByKey(1)(_+_)。
2. 若有多个分区，saveAsTextFile在这会输出多个文件，若要生成一个文件且文件不大可以使用repartition(1)控制分区，以便只输出一个文件。
3. 接下去可以使用Spark Streaming，满足实时计算。
