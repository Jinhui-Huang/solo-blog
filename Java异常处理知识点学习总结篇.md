


@[TOC](文章目录)

---

# 前言
Java异常是指在程序运行过程中发生的不正常情况，例如除数为零、数组越界、文件不存在等。Java提供了一套异常处理机制，通过使用try-catch-finally语句块来捕获和处理异常。
本文将介绍Java异常的基本概念和用法，包括以下几个方面：

>异常的分类：检查型异常和非检查型异常的区别和特点
异常的处理：try-catch-finally语句块的作用和用法
异常的抛出：throw关键字和throws关键字的作用和用法
异常的自定义：如何创建和使用自定义异常类
异常的特殊类：Error和AssertionError的含义和用法
异常的新特性：try-with-resources语句和多重catch语句的作用和用法
异常的原则和最佳实践：如何避免常见的异常处理错误和提高代码质量
---


# 一、异常的分类
Java异常可以分为两大类：检查型异常（checked exception）和非检查型异常（unchecked exception）。检查型异常是指编译器要求必须处理的异常，例如IOException、SQLException等。非检查型异常是指编译器不强制要求处理的异常，例如RuntimeException及其子类，如NullPointerException、ArithmeticException等。

- 检查型异常通常是由外部因素导致的，例如文件不存在、网络中断、数据库连接失败等。这些异常是可以预见并且可以恢复的，因此编译器要求程序员必须对这些异常进行处理，要么使用try-catch语句块捕获并处理，要么使用throws关键字声明并抛出，让调用者来处理。如果不处理检查型异常，编译器会报错。

- 非检查型异常通常是由程序逻辑错误导致的，例如除数为零、数组越界、空指针等。这些异常是可以避免并且不应该发生的，因此编译器不强制要求程序员处理这些异常，而是让它们在运行时抛出，并且终止程序。当然，程序员也可以选择捕获和处理这些异常，但一般不建议这样做，因为这样会掩盖程序错误。

# 二、异常的处理
Java提供了一套异常处理机制，通过使用try-catch-finally语句块来捕获和处理异常。try语句块包含可能发生异常的代码，catch语句块用于捕获特定类型的异常并进行处理，finally语句块用于无论是否发生异常都要执行的代码，例如释放资源等。try-catch-finally语句块可以嵌套使用，也可以在方法声明中使用throws关键字来抛出异常，让调用者来处理。
## 1.try-catch语句

try-catch语句是最基本的异常处理语句，它包含一个try语句块和一个或多个catch语句块。try语句块中的代码可能会发生某种类型或多种类型的异常，如果发生了异常，那么程序会跳出try语句块，并且寻找匹配的catch语句块来捕获并处理这个异常。如果没有匹配的catch语句块，那么这个异常会继续向上抛出，直到被捕获或者终止程序。

```java
try {
  // some code that may throw exceptions
} catch (ExceptionType1 e) {
  // handle exception type 1
} catch (ExceptionType2 e) {
  // handle exception type 2
} catch (ExceptionType3 e) {
  // handle exception type 3
}
```

在上面的例子中，try语句块中的代码可能会抛出三种类型的异常：ExceptionType1、ExceptionType2和ExceptionType3。如果发生了ExceptionType1，那么第一个catch语句块会捕获并处理它；如果发生了ExceptionType2，那么第二个catch语句块会捕获并处理它；如果发生了ExceptionType3，那么第三个catch语句块会捕获并处理它。如果发生了其他类型的异常，或者没有发生异常，那么这些catch语句块都不会执行。

多个catch语句块的顺序很重要，必须从最具体的异常类型开始，到最通用的异常类型结束，否则会导致编译错误。例如：

```java
try {
  // some code that may throw exceptions
} catch (FileNotFoundException e) {
  // handle file not found exception
} catch (IOException e) {
  // handle other input/output exception
} catch (Exception e) {
  // handle any other exception
}
```

在上面的例子中，如果将Exception放在第一个catch语句块，那么后面的catch语句块就永远不会执行，因为Exception可以捕获任何类型的异常，包括FileNotFoundException和IOException。这样就违反了多个catch语句块的原则，编译器会报错。

## 2.finally语句

finally语句是一个可选的语句块，用于在try-catch语句之后执行一些无论是否发生异常都要执行的代码，例如释放资源、关闭连接、清理缓存等。finally语句块的特点是它总是会执行，除非程序在try或者catch语句块中终止了（例如使用System.exit()方法）。例如：

```java
try {
  // some code that may throw exceptions
} catch (Exception e) {
  // handle exception
} finally {
  // always execute this code
}
```

在上面的例子中，无论try语句块是否发生异常，或者catch语句块是否处理了异常，finally语句块都会执行。如果try或者catch语句块中有return语句，那么finally语句块会在return之前执行。例如：

```java
public int test() {
  try {
    // some code that may throw exceptions
    return 1;
  } catch (Exception e) {
    // handle exception
    return 2;
  } finally {
    // always execute this code
    System.out.println("finally");
  }
}
```

在上面的例子中，无论test方法返回1还是2，都会先打印出"finally"。需要注意的是，如果finally语句块中也有return语句，那么它会覆盖try或者catch语句块中的return值，应该避免这种不好的编程习惯。
# 三、异常的抛出
Java提供了两个关键字来抛出异常对象：throw和throws。throw关键字用于在代码中手动抛出一个异常对象，通常用于自定义异常类或者在特定条件下触发异常。throws关键字用于在方法声明中指定该方法可能抛出的异常类型，让调用者来处理。
## 1.throw关键字
throw关键字用于在代码中手动抛出一个异常对象，通常用于自定义异常类或者在特定条件下触发异常。throw关键字后面必须跟一个异常对象，可以是Java提供的异常类的实例，也可以是自定义的异常类的实例。例如：

```java
public void setAge(int age) {
  if (age < 0 || age > 150) {
    throw new InvalidAgeException(age);
  }
  this.age = age;
}
```
在上面的例子中，我们在setAge方法中检查了年龄是否合法，如果不合法，就抛出一个InvalidAgeException对象。这个对象是我们自定义的一个非检查型异常类的实例，继承了RuntimeException类。这样，调用者就可以捕获和处理这个异常，或者让它继续向上抛出。
## 2.throws关键字
throws关键字用于在方法声明中指定该方法可能抛出的异常类型，让调用者来处理。throws关键字后面必须跟一个或多个异常类型，用逗号隔开。如果一个方法可能抛出多种类型的异常，那么它可以使用throws关键字来声明这些异常类型，而不需要在方法体中使用try-catch语句块来捕获和处理。例如：

```java
public void readFile(String fileName) throws FileNotFoundException, IOException {
  // some code that may throw file not found or input/output exception
}
```

在上面的例子中，我们在readFile方法声明中使用了throws关键字，指定了该方法可能抛出的两种类型的检查型异常：FileNotFoundException和IOException。这样，我们就不需要在方法体中使用try-catch语句块来捕获和处理这些异常，而是让调用者来处理。如果调用者也不处理这些异常，那么它也需要使用throws关键字来声明这些异常类型，或者使用try-catch语句块来捕获和处理。
# 四、异常的自定义
有时候，Java提供的异常类不能满足我们的业务需求，我们需要自定义一些异常类来表示特定的错误情况。自定义异常类需要继承Exception或者RuntimeException类，并提供构造方法和其他方法。例如：

```java
public class InvalidAgeException extends RuntimeException {
  private int age;

  public InvalidAgeException(int age) {
    super("Invalid age: " + age);
    this.age = age;
  }

  public int getAge() {
    return age;
  }
}
```

在上面的例子中，我们自定义了一个InvalidAgeException类，用于表示年龄不合法的情况。这个类继承了RuntimeException类，因为它是一个非检查型异常，不需要强制处理。我们在构造方法中调用了父类的构造方法，传入了一个异常信息，也可以传入一个异常原因或者其他参数。我们还提供了一个getAge方法，用于获取不合法的年龄值。

自定义异常类可以根据不同的业务需求来定义不同的类型和信息，方便程序员进行调试和处理。我们可以在代码中使用throw关键字来手动抛出自定义异常对象，例如：

```java
public void setAge(int age) {
  if (age < 0 || age > 150) {
    throw new InvalidAgeException(age);
  }
  this.age = age;
}
```

在上面的例子中，我们在setAge方法中检查了年龄是否合法，如果不合法，就抛出一个InvalidAgeException对象。这样，调用者就可以捕获和处理这个异常，或者让它继续向上抛出。

# 五、异常的特殊类
除了上述的异常处理机制，Java还提供了一些特殊的异常类，例如Error和AssertionError。Error是指程序无法处理的严重错误，例如OutOfMemoryError、StackOverflowError等。Error一般不需要捕获和处理，而是直接终止程序。AssertionError是指程序中的断言失败，例如assert x > 0;。断言是一种用于检查程序逻辑的工具，可以在开发和测试阶段开启，但在生产环境中关闭。断言失败会抛出AssertionError，也会终止程序。
## 1.Error类
Error类是Throwable类的子类，表示程序无法处理的严重错误，例如OutOfMemoryError、StackOverflowError等。这些错误通常是由虚拟机或者系统层面导致的，例如内存不足、栈溢出、链接错误等。这些错误是不可预见并且不可恢复的，因此一般不需要捕获和处理，而是直接终止程序。

Error类及其子类都是非检查型异常，编译器不强制要求处理它们。当然，程序员也可以选择捕获和处理它们，但一般不建议这样做，因为这样可能会导致程序处于不稳定的状态，或者掩盖真正的问题。例如：

```java
try {
  // some code that may throw error
} catch (OutOfMemoryError e) {
  // handle out of memory error
}
```

在上面的例子中，我们试图捕获和处理OutOfMemoryError，但这样可能会使得程序无法正常运行，或者忽略了内存泄漏或者其他问题。一般来说，当发生Error时，最好的做法是记录日志，并且尽快终止程序。

## 2.AssertionError类
AssertionError类是Error类的子类，表示程序中的断言失败，例如assert x > 0;。断言是一种用于检查程序逻辑的工具，可以在开发和测试阶段开启，但在生产环境中关闭。断言失败会抛出AssertionError，也会终止程序。

断言可以用于验证程序中的假设或者前提条件，例如参数有效性、循环不变式、结果正确性等。如果断言失败，说明程序中存在逻辑错误或者不一致性，需要修复或者调试。断言可以使用assert关键字来定义，有两种形式：

assert condition; 表示如果condition为false，则抛出AssertionError。
assert condition : message; 表示如果condition为false，则抛出AssertionError，并且附带一个message作为异常信息。
例如：

```java
public void divide(int x, int y) {
  assert y != 0 : "y cannot be zero";
  // some code to perform division
}
```

在上面的例子中，我们使用了断言来检查y是否为零，如果为零，则抛出AssertionError，并且附带一个
"y cannot be zero"作为异常信息。断言默认是关闭的，在运行时需要使用-ea或者-enableassertions参数来开启。例如：

>java -ea MyClass

在上面的命令中，我们使用-ea参数来运行MyClass类，并且开启断言功能。如果不使用这个参数，则断言不会执行。断言只应该用于开发和测试阶段，用于验证程序逻辑是否正确。在生产环境中应该关闭断言功能，避免影响性能或者安全性。

# 六、异常的新特性
从Java 7开始，Java引入了一些新的异常处理特性，例如try-with-resources语句和多重catch语句。这些特性可以简化异常处理的代码，也可以提高异常处理的效率和可读性。

## 1.try-with-resources语句
try-with-resources语句是从Java 7开始引入的一种新的异常处理语法，用于自动管理需要关闭的资源，例如文件、流、数据库连接等。try-with-resources语句的格式如下：

```java
try (Resource resource = new Resource()) {
  // use the resource
} catch (Exception e) {
  // handle exception
}
```

在上面的例子中，Resource是一个实现了AutoCloseable接口的类，表示它可以自动关闭。在try-with-resources语句中，可以声明并初始化一个或多个这样的资源，用分号隔开。当try语句块结束时，无论是否发生异常，这些资源都会被自动关闭，调用它们的close方法。这样可以避免在finally语句块中手动关闭资源，简化了代码，也避免了因为忘记关闭资源而导致的内存泄漏或者其他问题。

如果在try-with-resources语句中发生了异常，那么这些异常会被抛出，并且可以使用catch语句块来捕获和处理。如果在关闭资源时也发生了异常，那么这些异常会被抑制（suppressed），并且可以使用Throwable类的getSuppressed方法来获取。例如：

```java
try (FileInputStream fis = new FileInputStream("test.txt")) {
  // read from the file
} catch (IOException e) {
  // handle io exception
  Throwable[] suppressed = e.getSuppressed();
  // handle suppressed exceptions
}
```

在上面的例子中，如果在读取文件时发生了IOException，那么这个异常会被抛出，并且可以在catch语句块中处理。如果在关闭文件输入流时也发生了IOException，那么这个异常会被抑制，并且可以通过e.getSuppressed方法来获取一个Throwable数组，表示所有被抑制的异常。

## 2.多重catch语句
多重catch语句是从Java 7开始引入的一种新的异常处理语法，用于简化多个catch语句块的代码，当这些catch语句块有相同的处理逻辑时。多重catch语句的格式如下：

```java
try {
  // some code that may throw exceptions
} catch (ExceptionType1 | ExceptionType2 | ExceptionType3 e) {
  // handle these exception types with the same logic
}
```

在上面的例子中，我们使用|符号来分隔多个异常类型，表示这些异常类型可以使用同一个处理逻辑。这样可以减少代码的重复和冗余，也可以提高代码的可读性。例如：

```java
try {
  // some code that may throw exceptions
} catch (FileNotFoundException | UnknownHostException e) {
  // handle file not found or unknown host exception
} catch (Exception e) {
  // handle any other exception
}
```

在上面的例子中，我们使用多重catch语句来捕获和处理FileNotFoundException或者UnknownHostException，而不需要写两个相同的catch语句块。

# 七、异常链

异常链是指一个异常对象包含了另一个异常对象的引用，表示这两个异常之间有因果关系。异常链可以帮助程序员追踪和定位异常的根源，也可以提供更多的异常信息。异常链可以使用Throwable类的initCause方法或者带有Throwable参数的构造方法来创建。例如：

```java
public void connect(String url) throws ConnectionException {
  try {
    // some code that may throw MalformedURLException or IOException
  } catch (Exception e) {
    // create a new exception with the original exception as the cause
    throw new ConnectionException("Failed to connect to " + url, e);
  }
}
```

在上面的例子中，我们在connect方法中捕获了可能发生的MalformedURLException或者IOException，并且抛出了一个自定义的ConnectionException，表示连接失败。我们在创建ConnectionException对象时，传入了原始的异常对象作为参数，表示它是连接失败的原因。这样，我们就创建了一个异常链，可以通过ConnectionException对象的getCause方法来获取原始的异常对象，也可以通过printStackTrace方法来打印出完整的异常栈信息。

# 八、异常处理的原则和最佳实践
Java异常处理机制的目的是为了提高程序的健壮性和可维护性，让程序能够在发生异常时进行恢复或者提示，而不是直接崩溃。Java异常处理机制也遵循一些原则和最佳实践，例如：

- 避免使用异常来控制程序流程。异常应该只用于表示不正常或者错误的情况，而不应该用于实现正常的业务逻辑。使用异常来控制程序流程会降低程序的性能和可读性，也会使得程序难以调试和维护。
- 避免忽略或者屏蔽异常。如果捕获了一个异常，那么应该对它进行合理的处理，例如记录日志、抛出新的异常、恢复状态等。如果不处理异常，那么应该让它继续向上抛出，而不是将它忽略或者吞掉。忽略或者屏蔽异常会导致程序出现隐蔽的错误或者不一致性，也会使得程序难以调试和维护。
- 避免创建不必要的异常对象。创建和抛出异常对象会消耗系统资源和性能，因此应该尽量避免创建不必要的异常对象。例如，在循环中不要使用throw关键字来抛出相同的异常对象，而是在循环外面抛出；在catch语句块中不要使用new关键字来创建新的异常对象，而是使用原始的异常对象或者使用initCause方法来设置原因；在自定义异常类中不要重写fillInStackTrace方法，而是直接返回this等。
- 使用合适的异常类型和信息。当抛出一个异常时，应该使用合适的异常类型和信息，以便于程序员或者用户理解和处理这个异常。如果Java提供的异常类不能满足需求，可以自定义一个新的异常类，并且继承合适的父类。如果需要抛出一个检查型异常，应该在方法声明中使用throws关键字来指定这个异常类型。如果需要抛出一个非检查型异常，应该在文档中说明这个可能发生的情况。如果需要抛出一个包含原因的异常，应该使用initCause方法或者带有Throwable参数的构造方法来创建这个异常对象。
- 异常信息应该尽量清晰和具体，包含发生异常的原因、位置和影响，也可以包含建议的解决方案或者联系方式。

---

# 总结
Java异常是指在程序运行过程中发生的不正常情况，Java提供了一套异常处理机制，通过使用try-catch-finally语句块来捕获和处理异常。本文介绍了Java异常的基本概念和用法，包括异常的分类、处理、抛出、自定义、特殊类、新特性等，也介绍了一些异常处理的原则和最佳实践。希望本文能够帮助读者理解和掌握Java异常处理机制，提高程序的健壮性和可维护性。

