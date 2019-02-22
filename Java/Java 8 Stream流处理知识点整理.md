# Java 8 Stream流处理知识点整理

参考文档

[Java 8 Stream Tutorial](https://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/)

[winterbe/java8-tutorial](https://github.com/winterbe/java8-tutorial#streams)



## stream流处理

`java.util.Stream`类代表的是一组可以用一个或多个operation（操作）来执行的有序元素。filter()、map()、sorted()、foreach()等都是operation。operation可以分为intermediate（中间操作）和terminal（终端操作）。

通过stream处理可以简化代码，之前用若干行才能实现的功能可以缩减为一行。、



##### 创建一个stream

使用stream()创建：

```java
Arrays.asList("a1", "a2", "a3")
    .stream()
    .findFirst()
    .ifPresent(System.out::println);  // a1
```

使用Stream.of()创建：

```java
Stream.of("a1", "a2", "a3")
    .findFirst()
    .ifPresent(System.out::println);  // a1
```

创建一个1、2、3的stream：

```
IntStream.range(1, 4)
    .forEach(System.out::println);

// 1
// 2
// 3
```



```java
List<String> stringCollection = new ArrayList<>();
stringCollection.add("ddd2");
stringCollection.add("aaa2");
stringCollection.add("bbb1");
stringCollection.add("aaa1");
stringCollection.add("bbb3");
stringCollection.add("ccc");
stringCollection.add("bbb2");
stringCollection.add("ddd1");
```



##### Map

`map`即映射，可以将每个元素通过已有的方法转变为对应的其他元素

example1：

把每个string变成大写

```java
stringCollection
    .stream()
    .map(String::toUpperCase)
    .sorted((a, b) -> b.compareTo(a))
    .forEach(System.out::println);

// "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "AAA2", "AAA1"

```



example2:

将每个元素乘2后加一

```
Arrays.stream(new int[] {1, 2, 3})
    .map(n -> 2 * n + 1)
    .average()
    .ifPresent(System.out::println);  // 5.0
```



##### Match

`match`属于terminal operation，返回一个boolean值

anyMatch：

```java
boolean anyStartsWithA =
    stringCollection
        .stream()
        .anyMatch((s) -> s.startsWith("a"));

System.out.println(anyStartsWithA);      // true
```

allMatch：

```java
boolean allStartsWithA =
    stringCollection
        .stream()
        .allMatch((s) -> s.startsWith("a"));

System.out.println(allStartsWithA);      // false
```

noneMatch：

```java
boolean noneStartsWithZ =
    stringCollection
        .stream()
        .noneMatch((s) -> s.startsWith("z"));

System.out.println(noneStartsWithZ);      // true
```



##### Count

计算stream的元素数量，返回一个long值

```java
long startsWithB =
    stringCollection
        .stream()
        .filter((s) -> s.startsWith("b"))
        .count();

System.out.println(startsWithB);    // 3
```



##### Filter

`filter()`可以根据条件来删选出符合条件的元素

筛选出以“a”为开头的字符串元素：

```java
stringCollection
    .stream()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);

// "aaa2", "aaa1"
```



##### Collect

`collect（）`



#### Stream的处理顺序

先进行`filter（）`操作可以减少许多没有必要的操作

example：

先map后filter：

```java
Stream.of("d2", "a2", "b1", "b3", "c")
    .map(s -> {
        System.out.println("map: " + s);
        return s.toUpperCase();
    })
    .filter(s -> {
        System.out.println("filter: " + s);
        return s.startsWith("A");
    })
    .forEach(s -> System.out.println("forEach: " + s));

// map:     d2
// filter:  D2
// map:     a2
// filter:  A2
// forEach: A2
// map:     b1
// filter:  B1
// map:     b3
// filter:  B3
// map:     c
// filter:  C
```

先filter后map：

```java
Stream.of("d2", "a2", "b1", "b3", "c")
    .filter(s -> {
        System.out.println("filter: " + s);
        return s.startsWith("a");
    })
    .map(s -> {
        System.out.println("map: " + s);
        return s.toUpperCase();
    })
    .forEach(s -> System.out.println("forEach: " + s));

// filter:  d2
// filter:  a2
// map:     a2
// forEach: A2
// filter:  b1
// filter:  b3
// filter:  c
```





