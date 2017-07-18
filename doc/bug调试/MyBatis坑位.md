#MyBatis的坑
以下使用的MyBatis版本有可能不同，基本均在3.0版本以上。

###1. &lt;if test="value&lt;0">&lt;/if>

在xml中写sql不可避免用到if标签，而其中有个隐藏的要求
```    
与元素类型 "if" 相关联的 "test" 属性值不能包含 '<' 字符.
```
当使用&lt;if test="value&lt;0">&lt;/if>时，MyBatis解析会出错。百度到说是编码原因也可以排除。
进行查询实践之后发现使用&amp;lt;可以解决问题，那就说明xml解析时对属性里的“&lt;”进行了特殊字符的处理，导致报错。

###2. &lt;if test="value=='a'">&lt;/if>
在if标签的test里面，判断字符相等有关注点：
  '' 和 ""

设置整型value为97：
>&lt;if test="value=='a'">#{value}&lt;/if>  =>  97
&lt;if test='value=="a"'>#{value}&lt;/if>  =>  抛出异常：java.lang.NumberFormatException: For input string: "a"
ps:均以map方式传入参数

设置String类型value为"a"：
>&lt;if test="value=='a'">#{value}&lt;/if>  =>  抛出异常：java.lang.NumberFormatException: For input string: "a"
&lt;if test='value=="a"'>#{value}&lt;/if>  =>  a
ps:均以map方式传入参数

可以看出：
单引号的字符会被看做整数，双引号的对应String类型，当传入的参数类型不能对应时，抛出NumberFormatException异常。
>    ps:在test属性里面使用=可以赋值。

