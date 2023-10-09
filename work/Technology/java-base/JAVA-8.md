## Lambda

主要用以实现函数式编程（更加灵活）

主要优点：

1. 性能开销（不需要使用匿名类进行预先加载）
2. 未来改变的可能性，因为是动态加载的 所以更容易在程序中实现动态修改

##### 基本概念

使用的 ( 参数 ) -> { 函数体 } 表达式是 JAVA 语言中的语法糖，实际相当于创建一个 function 包下的对应的对象

常见 @FuncationalInterface ： Function<input-type, output-type> Predicate<T> Comparator ...

所有函数体都是通过注解 @FunctionalInterface 实现，并且类只能有接口**只能有一个抽象方法**

##### 实际使用

1.  e -> { System.out.println(e) }
2.  System.out::println
3.  identity() 表示对象本身

## Stream API

Stream 是对集合操作的一个抽象类，用以实现对集合的基础操作

常见方法：

filter sorted map distinct 

skip limit count 

anyMatch allMatch findFirst findAny 

reduce .....

**为基础类型的数组创建了  IntStream LongStream 等**

## Collectors