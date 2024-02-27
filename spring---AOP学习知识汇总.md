


# 一. AOP简介

- AOP(Aspect Oriented Programming)面向切面编程, 一种编程范式, 指导开发者如何组织程序结构

- OOP(Object Oriented Programming)面向对象编程

AOP作用: 在不惊动原始设计的基础上为其进行功能增强

Spring的理念: 无侵入式\无入侵式编程

# 二. AOP核心概念

![](AOP核心概念.PNG)
图片来自B站黑马程序员([SSM-AOP简介](https://www.bilibili.com/video/BV1Fi4y1S7ix?p=31&vd_source=b1a441fcb369fd950d8bf49580ca3248))

- 通知(Advice): 所要增加功能的方法
- 切入点(Pointcut): 需要增加该功能的方法, 必须定义成切入点
- 连接点(JoinPoint): 可以增加该功能的方法, 即程序中任意位置的方法, 连接点包含着切入点
- 通知类: 承载通知方法的类, 在切入点处执行的操作, 也就是共性功能
- 切面(Aspect): 连接通知和切入点之间的桥梁, 描述两者之间的关系

# 三. AOP入门案例

开发模式: 注解

坐标导入:

```xml

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.20</version>
    </dependency>

    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.6</version>
    </dependency>
</dependencies>

```

编写共性功能(通知类与通知);

```java
/*
 * 通知类
 * */
public class MyAdvice {

    /*
     * 通知
     * */
    public void Method() {
        System.out.println(System.currentTimeMillis());
    }
}
```

定义切入点(依然写在通知类里):

```java
/*
 * 通知类
 * */
public class MyAdvice {
    /*
     * 描述切入点@Pointcut("execution(void com.itstudy.dao.BookDao.update())")
     * @Point("execution(切入点方法返回类型 切入点包名.类名.方法名)")
     * 切入点定义依托在一个不具有实际意义的方法上进行, 即无参数, 无返回值, 方法体无实际逻辑
     * */
    @Pointcut("execution(void com.itstudy.dao.BookDao.update())")
    private void pt() {
    }

    /*
     * 通知
     * */
    public void Method() {
        System.out.println(System.currentTimeMillis());
    }
}
```

绑定切入点和通知(切面), 在通知的上方描述切面, 同时声明通知类和受到spring控制:

```java

@Component
@Aspect
public class MyAdvice {
    /*
     * 描述切入点@Pointcut("execution(void com.itstudy.dao.BookDao.update())")
     * @Point("execution(切入点方法返回类型 切入点包名.类名.方法名)")
     * */
    @Pointcut("execution(void com.itstudy.dao.BookDao.update())")
    private void pt() {
    }

    /*
     * 通知(共性功能)
     *
     * 切面描述@Before("pt()")
     * 在切入点执行之前执行下面这个方法
     * */
    @Before("pt()")
    public void Method() {
        System.out.println(System.currentTimeMillis());
    }
}
```

配置文件里SpringConfing也要注解上 声明有注解开发的AOP, 即启动了AOP

```java

@Configuration
@ComponentScan("com.itstudy")
@EnableAspectJAutoProxy
public class SpringConfig {

}
```

进行测试类测试

```java
public class App {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        BookDao bookDao = ctx.getBean(BookDao.class);
        bookDao.update();
    }
}
```

没有AOP之前, 就一条语句

```
book dao update...
```

注解开发AOP后, update方法有了新功能

```
1685956645269
book dao update...
```

# 四. AOP工作流程

1. Spring容器启动
2. 读取所有切面配置中的切入点
3. 初始化bean, 判定bean对应的类中的方法是否匹配到任意切入点
    - 匹配失败, 创建对象
    - 匹配成功, 创建原始对象(**目标对象**)的**代理对象**
    - `System.out.println(bookDao.getClass())` 出来的结果是代理对象`class com.sun.proxy.$Proxy20`
4. 获取bean执行方法
    - 获取bean, 调用方法并执行, 完成操作
    - 获取的bean是代理对象时, 根据代理对象的运行模式运行原始方法与增强的内容, 完成操作

SpringAOP本质: 代理模式

# 五. AOP切入点表达式

描述方式一: 执行com.itstudy.dao包下的BookDao接口中的无参数update方法

`@Pointcut("execution(void com.itstudy.dao.BookDao.update())")`

描述方式二: 执行com.itstudy.dao,impl包下的BookDaoImpl类中的无参数update方法

`@Pointcut("execution(void com.itstudy.dao.impl.BookDaoImpl.update())")`

## 1. 语法格式

标准格式: 动作关键字(访问修饰符 返回值类型 包名.类/接口名.方法名(参数) 异常名)

`execution(public User com.itstudy.service.UserService.fingdById(int))`

- 动作关键字execution: 执行
- 访问修饰符: public(可以省略)
- 返回值类型: User
- 包名: com.itstudy.service
- 类/接口名: UserService
- 方法名: fingdById
- 参数类型: int
- 异常名: 可以省略

## 2. 通配符

- \* :单个独立的任意符号, 也可以独立出现, 也可以作为前缀或者后缀的匹配符出现

`execution(public * com.itstudy.*.UserService.find*(*))`

匹配com.itstudy包下任意包中的UserService类或接口中所有find开头的带有一个参数的方法

- .. :多个连续的任意符号, 可以独立出现,常用于简化包名与参数的书写

`execution(public User com..UserService.findById(..))`

匹配com包下的任意包中的UserService类或接口中所有名称为findById的方法

- \+ :专用于匹配子类类型(了解即可)

`execution(public * *..*Service+.*(..))`

## 3. 书写技巧

- 所有代码按照规范开发, 否则以下技巧失效
- 描述切入点通常描述接口, 而不描述实现类
- 访问控制修饰符针对接口开发均采用public描述(可省略)
- 返回值类型对于增删改查使用精准类型加速匹配, 对于查询类使用*通配快速描述
- 包名书写经量不使用..匹配, 效率过低, 常用*做单个包描述匹配, 或精准匹配
- 接口名/类名书写名称与模块相关的采用匹配, 例如UserService书写成*Service, 绑定业务层接口名
- 方法名书写以动词进行精准匹配, 名词采用*匹配, 例如getById书写成getBy*, selectAll书写成selectAll
- 参数规则较为复杂, 根据业务方法灵活调整
- 通常不使用异常作为匹配规则

# 六. AOP通知类型

AOP通知描述了抽取的共性功能, 根据功能抽取的位置不同, 最终运行代码时要将其加入到合理的位置

共五种类型:

- 前置通知@Before("pt()")
- 后置通知@After("pt()")
- 环绕通知(重点)@Around("pt2()")
- 返回后通知(了解)@AfterReturning("pt()")
- 抛出异常后通知(了解)@AfterThrowing("pt2()")

重点说明@Around() 环绕通知

```java

@Component
@Aspect
public class MyAdvice {
    /*
     * 描述切入点@Pointcut("execution(void com.itstudy.dao.BookDao.update())")
     * @Point("execution(切入点方法返回类型 切入点包名.类名.方法名)")
     * */
    @Pointcut("execution(void com.itstudy.dao.BookDao.update())")
    private void pt() {
    }

    //@Around("pt()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("around before advice...");
        //表示对原始操作的调用, 会强制抛异常
        //环绕通知必须依赖形参ProceedingJoinPoint才能对原始方法调用
        //必须抛出异常Throwable, 因为原方法中说不定会有异常
        Object proceed = pjp.proceed();
        System.out.println("around after advice...");
        return proceed;
    }
}
```

1. 环绕通知必须依赖形参ProceedingJoinPoint才能对原始方法调用
2. 通知必须抛出异常Throwable, 因为原方法中说不定会有异常
3. 返回值建议书写, 尽管原方法是void, Object proceed = pjp.proceed()会获取原方法的返回值, 可以return出去, 也可以return一个新值出去

# 七. AOP通知获取数据

原始方法含有参数和返回值, 下面将演示AOP如何获取这些

```java

@Repository
public class BookDaoImpl implements BookDao {
    @Override
    public int findName(String name) {
        System.out.println("name: " + name);
        return 666;
    }
}
```

## 1. 获取参数

所有通知类型都可以

以@Befor()为例

```java

@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itstudy.dao.BookDao.*(..))")
    private void pt() {
    }

    @Before("pt()")
    public void beforeFindName(JoinPoint jp) {
        //获取参数
        Object[] args = jp.getArgs();
        System.out.println(Arrays.toString(args));
        System.out.println("before advice findName...");
    }
}
```

对于@Around(), ProceedingJoinPoint继承了JoinPoint, 自然可以拿到参数

```java

@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itstudy.dao.BookDao.*(..))")
    private void pt() {
    }

    @Around("pt()")
    public Object countTime(ProceedingJoinPoint pjp) throws Throwable {
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        args[0] = "alen";

        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            pjp.proceed(args);
        }
        long end = System.currentTimeMillis();
        System.out.println("运行了" + (end - start) + "ms");
        return pjp.proceed(args);
    }
}
```

这样就可以拿到原始方法传进来的参数, 也可以直接更改原始参数再传回原始方法, 结果显示参数发生了更改

```
[tom]
name: alen
name: alen
...
name: alen
666
```

## 2. 获取返回值

只有返回后通知和环绕通知可以获取

需要更改注解的内容, 定义一个接收返回值的形参, 方法里也要定义同名的接收返回值的形参

```java

@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itstudy.dao.BookDao.*(..))")
    private void pt() {
    }

    @AfterReturning(value = "pt()", returning = "ret")
    public void afterReturning(Object ret) {
        System.out.println("afterReturning advice..." + ret);
    }
}
```

运行结果:

```
afterReturning advice...666
```

注意: 接收参数和返回值的形参是有顺序的`method(JoinPoint jp, Object ret)`

## 3. 获取异常
只有抛出异常后通知和环绕通知可以
对于抛出异常后通知与获取返回值类型类似
```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itstudy.dao.BookDao.*(..))")
    private void pt() {
    }
    @AfterThrowing(value = "pt()", throwing = "throwable")
    public void afterThrowing(Throwable throwable) {
        System.out.println("afterThrowing advice..." + throwable);
    }
}
```
原始方法加个异常测试`int i = 1/0;`

抛出异常结果为:
```
name: tom
afterThrowing advice...java.lang.ArithmeticException: / by zero
Exception in thread "main" java.lang.ArithmeticException: / by zero
	at com.itstudy.dao.impl.BookDaoImpl.findName(BookDaoImpl.java:30)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:566)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:344)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:198)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
	at org.springframework.aop.aspectj.AspectJAfterThrowingAdvice.invoke(AspectJAfterThrowingAdvice.java:64)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:215)
	at com.sun.proxy.$Proxy20.findName(Unknown Source)
	at App.main(App.java:17)
```

对于环绕通知直接捕获异常
```
try {
      pjp.proceed(args);
    } catch (Throwable e) {
        e.printStackTrace();
}
```

