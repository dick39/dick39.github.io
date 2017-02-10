#Java8 新特性学习使用

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

这中间有用到类型推断，函数式接口的其他特性，准备下次再继续写。

