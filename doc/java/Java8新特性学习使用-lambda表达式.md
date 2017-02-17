#Java8 新特性学习使用-lambda表达式
##java8新特性
在java8的众多更新中，最引人瞩目的无疑是lambda表达式与stream接口。接下来的学习记录本来是要做一次组内分享的，现在也就仅仅记录在这。

##lambda表达式
这是一函数式编程思想的实现方式，使代码更简洁，以解决匿名内部类最大的痛点——语法冗余。
先看一个简单控制台输出『hello world』的线程实现例子：
```java
new Thread(new Runnable(){
            @Override
            public void run() {
                System.out.println("hello world!");
            }
        }).start();
```
而使用lambda实现的相同功能：
```java
new Thread(()->System.out.println("hello world!")).start();
```
同时这样可以将一个lambda表达式传入一个方法，类似于回调。
ps:这中间有用到类型推断，函数式接口的其他特性，准备下次再继续写。

##作用域
在匿名内部类的实现方式中，想要调用外部参数，必须是显式声明为final。而在lambda表达式中，可以使用未声明为final的参数，只需要满足Effectively final，即事实上的不变参数。
```java
Integer i = 1;
Runnable r = ()->System.out.println(i);
//i=2;//若打开注释，编译会报错，Local variable s defined in an enclosing scope must be final or effectively final
```

下一章会记录一下其他特性，并结合运用

##引用
[深入理解Java 8 Lambda](http://www.cnblogs.com/figure9/archive/2014/10/24/4048421.html)
