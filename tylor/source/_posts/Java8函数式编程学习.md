---
title: Java8函数式编程学习
date: 2018-02-01 22:30:08
tags: JDK
category: blog
---
Java8中新增FunctionalInterface注解用来标记函数式接口，并且定义了4种基本函数式接口：
- Function
    * R apply(T t); 接收T类型参数，返回R类型结果
    * default compose(Function before) 参数前处理
    * default andThen(Function after) 结果后处理
- Predicate
    * boolean test(T t); 接收参数，返回布尔值
    * default and 逻辑与
    * default negate 逻辑非
    * default or 逻辑或
- Supplier
    * T get();
- Consumer
    * void accept(T t);
    * default andThen(Consumer after)

如果忽略 default方法的话，我们可以把function视为一个一元函数，单个输入，单个输出。
而Predicate则是固定结果类型为boolean的function，Supplier是一个无参函数，Consumer 无返回的函数。
其中，Function、Predicate、Consumer都有default方法支持连续式调用，而且都有Bi系列支持双参数。
另外，每种接口还有相应的基本类型接口。

示例
````java
BiFunction<String,String,Integer> add =
    (x,y)->Integer.parseInt(x)+Integer.parseInt(y);
Function<Integer,Integer>  square = x-> x*x;
Predicate<Integer> predicate = x-> x>100;
BiPredicate<String,String> biPredicate = (x,y)->{
    return predicate.test(add.andThen(square).apply(x,y));
};
System.out.println(biPredicate.test("5","6"));
````
这个例子没有加任何注释，但是应该还算通俗易懂。
相比于命令式编程，这种方式应该更符合我们的思维习惯。
通过函数式编程来定义运算的方式，而不再拘泥于过程中每一步对象的维护，
我们可以更多的关注、更清晰的体现运算逻辑，提升coding的体验和代码可读性。

写这个例子的过程，我甚至回想起了学生时代做数学题的感觉，如果现在的学生能写代码来解数学题，相信也是一种很有成就感的体验。