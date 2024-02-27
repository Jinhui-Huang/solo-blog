


@[TOC](文章目录)

---

# 前言

Java 字符串是一种常用的数据类型，它在程序中的作用不言而喻。但是，你是否了解 Java 字符串的内存原理呢？本文将从以下几个方面对 Java 字符串的内存原理进行分析：

1. 
2. Java 字符串的不可变性
3. Java 字符串的常量池
4. Java 字符串的拼接和优化

---

# 一、Java 字符串的存储结构
Java 字符串是由 char 数组和一个 int 值组成的对象，char 数组用于存储字符串的字符，int 值用于存储字符串的长度。在 Java 9 之前，char 数组是以 UTF-16 编码方式存储的，每个字符占用两个字节。在 Java 9 之后，为了节省内存空间，char 数组被替换为 byte 数组，并引入了一个编码标志位，用于表示字符串是以 UTF-16 还是 Latin-1 编码方式存储的，每个字符占用一个或两个字节。
![JDK17里对String类成员变量的描述，是一个字节数组](https://img-blog.csdnimg.cn/b41b3df4d7904cb09a8a5e2add2d707b.png#pic_center)

# 二、Java 字符串的不可变性

Java 字符串是不可变的，也就是说，一旦创建了一个字符串对象，它的内容就不能被修改。这样做有以下几个好处：

- 简化了字符串的操作和比较，不需要考虑字符串的状态变化
- 提高了字符串的安全性，避免了字符串被恶意修改或篡改
- 方便了字符串的缓存和共享，减少了内存开销和垃圾回收压力


但是，字符串的不可变性也带来了一些问题，例如：

- 每次修改字符串都会创建一个新的字符串对象，导致内存浪费和性能下降
- 需要使用特殊的类（如 StringBuilder 或 StringBuffer）来实现字符串的可变性和高效拼接
- 
# 三、Java 字符串的常量池
为了解决字符串对象的重复创建和内存浪费问题，Java 引入了一个特殊的区域，叫做字符串常量池（String Constant Pool），它是方法区（Method Area）的一部分，用于存储字符串常量。当创建一个字符串常量时，Java 会先检查字符串常量池中是否已经存在相同内容的字符串对象，如果存在，则直接返回该对象的引用；如果不存在，则在字符串常量池中创建一个新的字符串对象，并返回其引用。但如果是用new关键字创造出来的字符串并不是在常量池中，而是由new在堆内存新开辟的一处空间里。这里的地址值和串池里的地址值完全不同。

```java
String str1 = "abc"; //直接赋值在串池
        String str2 = new String("abc"); //new直接在堆内存中开辟新空间，地址值和串池里的地址值不一样
        String str3 = "abc"; //串池存在，直接复用，和str1地址值一样
        System.out.println(str1);
        System.out.println(str2);

        boolean flag1 = str1 == str2; //地址值不同
        System.out.println(flag1); //false

        boolean flag2 = str1 == str3; //地址值相同
        System.out.println(flag2);  //true
```

# 四、Java 字符串的拼接和优化

先来看Java中创建字符串并进行拼接的过程。
```java
String s1 = "Hello";
String s2 = "World";
String s3 = s1 + " " + s2; // s3 = "Hello World"
```

这种方式虽然看起来很简单，但实际上，在String类的底层代码上，每次使用 + 运算符，都会在堆内存中由new关键字创建一个新的字符串对象，并将原来的字符串复制到新的对象中。这样就会产生很多临时的字符串对象，占用内存空间，并增加垃圾回收的负担。如果我们要拼接很多个字符串，例如：

```java
String s = "";
for (int i = 0; i < 100; i++) {
s = s + i; // 每次循环都会创建一个新的字符串对象
}
```

那么这种方式就会非常低效，因为它会创建 100 个临时的字符串对象，并且每次都得复制之前拼接的结果。不仅运行效率慢，还会大大占用堆内存中的空间

为了解决这个问题，Java 提供了一个专门用于字符串拼接的类：StringBuilder。StringBuilder 是一个可变的字符序列，它可以动态地改变自己的长度和内容。我们可以用 append 方法来向 StringBuilder 中添加字符串，然后用 toString 方法来转换成最终的字符串。例如：

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 100; i++) {
sb.append(i); // 不会创建新的字符串对象，只是在原有的字符序列后面添加字符
}
String s = sb.toString(); // 最后转换成字符串
```

这种方式就会更高效，因为它只会创建一个 StringBuilder 对象，并且不需要复制之前拼接的结果。

除了使用 StringBuilder，我们还可以使用 StringJoiner 类来实现字符串拼接。StringJoiner 是一个专门用于连接多个字符串并添加分隔符和前缀后缀的类。我们可以用 add 方法来向 StringJoiner 中添加字符串，然后用 toString 方法来转换成最终的字符串。例如：

```java
StringJoiner sj = new StringJoiner(", ", "[", "]"); // 创建一个 StringJoiner 对象，并指定分隔符为逗号，前缀为左中括号，后缀为右中括号
for (int i = 0; i < 100; i++) {
sj.add(String.valueOf(i)); // 向 StringJoiner 中添加字符串
}
String s = sj.toString(); // 最后转换成字符串
```

这种方式可以方便地生成一些格式化的字符串，例如列表、数组、集合等。

总之，在 Java 中，如果我们需要进行大量的字符串拼接操作，我们应该避免使用 + 运算符，而是使用 StringBuilder 或 StringJoiner 等专门的类来优化性能和内存消耗。

---

# 总结
进行字符串比较时不用 ==  的原因首先是因为 == 在引用类型数据里比较的是地址值，而字符串会由于内容相同但是地址值不同给出false的结果，这也是从字符串的原理得出的，所以比较字符串还是选择String类的equals()方法。
