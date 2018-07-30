
---
title: Kotlin入门学习笔记（一）
date: 2017-05-19 15:58:35
tags:
---

![image](http://7xz8pr.com1.z1.glb.clouddn.com/kotlin.png)  

突然间，Kotlin被宣布成为Android开发第一语言。一下子就爆炸了，什么三年干掉Java，全民Kotlin。。。

![image](http://7xz8pr.com1.z1.glb.clouddn.com/mengbi.jpg)



好了，废话不多说，直接学习来入门吧：






-  ## 包声明

```
package my.demo

import java.util.*
```
　包不需要与目录匹配，可以任意放在系统文件中。
　
-  ## 方法定义
先上栗子

```
fun sum(a: Int, b: Int): Int {
    return a + b
}
```
以fun来表示函数，与js中的function有点类似哈，然后sum是方法名，里面的参数a，和定义参数类似Int都能看懂，后面跟着：Int就是返回是Int。

还可以更简洁的表达式：

```
fun sum(a: Int, b: Int) = a + b
```

方法返回一个没有意义的值，可以用Unit，也可以省略：

```
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}

fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
```

-  ## 局部变量

只能赋值一次（只读）的局部变量的几种声明方式：  
使用关键字` val `表示，可以看成java的` static final `和js中的` const `
```
val a: Int = 1  // immediate assignment
val b = 2   // `Int` type is inferred
val c: Int  // Type required when no initializer is provided
c = 3       // deferred assignment
```

上面栗子说明：  
1、直接声明变量类型和值  
2、类型可以省略，系统自动识别是Int  
3、先声明变量，和类型，在赋值。

###### 真正可变的局部变量

```
var x = 5 // `Int` type is inferred
x += 1
// x = 6
```

和js都是用` var `来声明哈。但是和js又有什么不同呢，这个坑以后填，嘿嘿。


-  ## 注释
和java、js一样也是使用` / / `和` /* */ `来注释，但是不同的是，块注释` /*  */ `可以嵌套。好像没什么说，过~


-  ## 使用字符串模板
简单的栗子：

```
fun main(args: Array<String>) {
    var a = 1
    // simple name in template:
    val s1 = "a is $a" 

    a = 2
    // arbitrary expression in template:
    val s2 = "${s1.replace("is", "was")}, but now is $a"
    println(s2)
    
    //a was 1, but now is 2
}
```

其中模板表达式由$符号开始，和一个值组成。
更详细的，比如运行结果是` a was 1, but now is 2  `，以后会说明。

-  ## 条件语句

这个基本使用和Java一样，还可以使用简洁表达式，给个栗子：

```
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}

fun maxOf(a: Int, b: Int) = if (a > b) a else b
```

-  ## 使用Null值与检查Null
特别指出如果一个值可能为空，必须显式地表示为空。

```
fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)

    // Using `x * y` yields error because they may hold nulls.
    if (x != null && y != null) {
        // x and y are automatically cast to non-nullable after null check
        println(x * y)
    }
    else {
        println("either '$arg1' or '$arg2' is not a number")
    }    
}

```

or


```
// ...
if (x == null) {
    println("Wrong number format in arg1: '${arg1}'")
    return
}
if (y == null) {
    println("Wrong number format in arg2: '${arg2}'")
    return
}

// x and y are automatically cast to non-nullable after null check
println(x * y)
```



上面栗子是传入Str，然后转成Int，但是Str不是数字，转换成Int就会是null，而不是出现像Java出现转换` java.lang.NumberFormatException `或者js出现` NaN `。

-  ## 类型检查与自动转换
` is `操作符会检查这个表达式是否是一个类型的实例。如果一个不可以变的常量或属性已经被检查过了，那么就先不需要显式的转换。


```
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
        // `obj` 在if里面自动转成 `String` 
        return obj.length
    }

    // `obj` 是`Any`类型返回null
    return null
}
```

or


```
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null

    // `obj` 在if外面自动转成 `String`
    return obj.length
}
```

or even


```
fun getStringLength(obj: Any): Int? {
    //`obj` 在&&右变自动转成 `String`
    if (obj is String && obj.length > 0) {
        return obj.length
    }

    return null
}
```
-  ## 使用for循环
这个类似Java的for each，但是对于js和Python来说肯定很熟悉了

```
val items = listOf("apple", "banana", "kiwi")
for (item in items) {
    println(item)
}
```

or

```
val items = listOf("apple", "banana", "kiwi")
for (index in items.indices) {
    println("item at $index is ${items[index]}")
}
```

上面栗子说明：
第二个栗子中：` indices `表示前面的` index `是items的下标0,1,2...  


-  ## 使用while循环
` while `使用和其他没什么区别
```
val items = listOf("apple", "banana", "kiwi")
var index = 0
while (index < items.size) {
    println("item at $index is ${items[index]}")
    index++
}
```

-  ## 使用when表达式
哈哈看到这里是不是瞬间懵逼了，里面的lambda表达式` -> `先不用看，就单单when使用也一样
```
fun describe(obj: Any): String =
when (obj) {
    1          -> "One"
    "Hello"    -> "Greeting"
    is Long    -> "Long"
    !is String -> "Not a string"
    else       -> "Unknown"
}
```
上面栗子：` describe `是一个` function `当传入` 1 `，返回` One `，依次类推。

-  ## 范围
`in`操作符，检测一个数字是否在某个范围里

```
val x = 10
val y = 9
if (x in 1..y+1) {
    println("fits in range")
}
```
可以使用`!`取反

```
val list = listOf("a", "b", "c")

if (-1 !in 0..list.lastIndex) {
    println("-1 is out of range")
}
if (list.size !in list.indices) {
    println("list size is out of valid list indices range too")
}

//-1 is out of range
//list size is out of valid list indices range too
```

和for循环配合：

```
for (x in 1..5) {
    print(x)
}
//12345
```

其中`..`表示从小到大，`1..10`，表示范围1到10。那么如果从大到小呢，那就是使用`downTo`，比如：`9 downTo 0`，表示从9到0。还可以使用`setp`，类似每次循环+几或者-几，是+或者-，看是`..`还是`downTo`。


```
for (x in 1..10 step 2) {
    print(x)
}
//13579

for (x in 9 downTo 0 step 3) {
    print(x)
}
//9630
```

-  ## 集合
前面也有是用过，但是关于集合的定义不做过多详细介绍，简单介绍怎么使用：

```
    val items = listOf("apple", "banana", "kiwi")
    for (item in items) {
        println(item)
    }
```

使用in操作符，判断集合是否包含某个`object`

```
    val items = setOf("apple", "banana", "kiwi")
    when {
        "orange" in items -> println("juicy")
        "apple" in items -> println("apple is fine too")
    }
```

使用lambda表达式来filter（过滤）和map（类似js中的map函数）集合：

```
    val fruits = listOf("banana", "avocado", "apple", "kiwi")
    fruits
    .filter { it.startsWith("a") }  //过滤以a开头
    .sortedBy { it }                //默认acs排序
    .map { it.toUpperCase() }       //转换成大写
    .forEach { println(it) }        //迭代输出
    
    //APPLE
    //AVOCADO
```

以上就是Kotlin的基础语法介绍与简单使用。  
我相信看完了，肯定有更多的疑问。。。期待更多的更新，估计是下一周了  
　　　　　　　　　　　　　　　　　　　　　　　　　　　　 ——by Leeves












