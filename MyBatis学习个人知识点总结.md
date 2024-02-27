<!-- TOC -->
* [一. Mybatis-入门](#一-mybatis-入门)
  * [1. 引入Mybatis和mysql.driver的相关依赖, 配置Mybatis](#1-引入mybatis和mysqldriver的相关依赖-配置mybatis)
  * [2. 编写SQL语句(注解/XML)](#2-编写sql语句注解xml)
  * [3. JDBC](#3-jdbc)
  * [4. MyBatis数据库连接池](#4-mybatis数据库连接池)
  * [5. lombok工具包](#5-lombok工具包)
* [二. MyBatis的基础操作](#二-mybatis的基础操作)
  * [1. 准备数据库](#1-准备数据库)
  * [2. 基础操作-删除](#2-基础操作-删除)
    * [(1). 根据主键ID删除](#1-根据主键id删除)
    * [(2). 根据主键ID批量删除](#2-根据主键id批量删除)
  * [3. 基础操作-新增](#3-基础操作-新增)
  * [4. 基础操作-更新](#4-基础操作-更新)
    * [根据主键id更新](#根据主键id更新)
  * [5. 基础操作-查询](#5-基础操作-查询)
    * [(1). 根据id查询](#1-根据id查询)
    * [(2). 条件查询](#2-条件查询)
* [三. MyBatis的XML映射文件配置SQL语句](#三-mybatis的xml映射文件配置sql语句)
  * [1. 配置XML文件](#1-配置xml文件)
  * [2. 动态SQL语句](#2-动态sql语句)
    * [(1). 动态SQL-if & where](#1-动态sql-if--where)
      * [相关案例, 动态更新员工信息](#相关案例-动态更新员工信息)
    * [(2). 动态SQL-foreach](#2-动态sql-foreach)
    * [(3). 动态SQL-sql & include](#3-动态sql-sql--include)
<!-- TOC -->

# 一. Mybatis-入门

## 1. 引入Mybatis和mysql.driver的相关依赖, 配置Mybatis

首先新建springboot项目, 然后在pom.xml文件下进行配置

```xml

<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>

<dependency>
<groupId>com.mysql</groupId>
<artifactId>mysql-connector-j</artifactId>
<version>8.0.33</version>
<scope>runtime</scope>
</dependency>

<dependency>
<groupId>mysql</groupId>
<artifactId>mysql-connector-java</artifactId>
<version>8.0.28</version>
</dependency>
```

## 2. 编写SQL语句(注解/XML)

首先编写一个User类来存放获取到的数据, 各个成员变量要和数据库表里的字段名一样

```java
public class User {
    private Integer customer_id;
    private String first_name;
    private String last_name;
    private String birth_date;
    private String phone;


    public User() {
    }

    public User(Integer customer_id, String first_name, String last_name, String birth_date, String phone) {
        this.customer_id = customer_id;
        this.first_name = first_name;
        this.last_name = last_name;
        this.birth_date = birth_date;
        this.phone = phone;
    }
}
```

详细请看[User.java](MybatisDemo01%2Fsrc%2Fmain%2Fjava%2Fcom%2Fitstudy%2Fpojo%2FUser.java)

然后配置mybatis的配置文件application.properties

```properties
# jdbc.driver
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# Database's url
spring.datasource.url=jdbc:mysql://localhost:3306/sql_store
# username
spring.datasource.username=mybatis
# password
spring.datasource.password=mybatis
```

新建一个接口来自动获取一个存放user列表的bean对象, 同时在里面注解定义sql语句

```java

@Mapper //在运行时, 会自动生成该接口的实现类对象(代理对象), 并且将该对象交给IoC容器管理
public interface UserMapper {
    //查询全部用户的信息
    @Select("select customer_id, first_name, last_name, birth_date, phone from customers " +
            "limit 10")
    List<User> list();

}
```

最后在测试类里进行bean的调用

```java

@SpringBootTest  //springboot整合单元测试的注解
class AppTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testListUser() {
        List<User> userList = userMapper.list();
        userList.stream().forEach(System.out::println);
    }
}
```

运行结果打印出了10条顾客的信息

```
User{customer_id=1, first_name='Babara', last_name='MacCaffrey', birth_date='1986-03-28', phone='781-932-9754'}
User{customer_id=2, first_name='Ines', last_name='Brushfield', birth_date='1986-04-13', phone='804-427-9456'}
User{customer_id=3, first_name='Freddi', last_name='Boagey', birth_date='1985-02-07', phone='719-724-7869'}
User{customer_id=4, first_name='Ambur', last_name='Roseburgh', birth_date='1974-04-14', phone='407-231-8017'}
User{customer_id=5, first_name='Clemmie', last_name='Betchley', birth_date='1973-11-07', phone='null'}
User{customer_id=6, first_name='Elka', last_name='Twiddell', birth_date='1991-09-04', phone='312-480-8498'}
User{customer_id=7, first_name='Ilene', last_name='Dowson', birth_date='1964-08-30', phone='615-641-4759'}
User{customer_id=8, first_name='Thacher', last_name='Naseby', birth_date='1993-07-17', phone='941-527-3977'}
User{customer_id=9, first_name='Romola', last_name='Rumgay', birth_date='1992-05-23', phone='559-181-3744'}
User{customer_id=10, first_name='Levy', last_name='Mynett', birth_date='1969-10-13', phone='404-246-3370'}
```

**以上信息均为随机生成的假信息**

## 3. JDBC

JDBC: 就是使用Java语言操作关系型数据库的一套API

各个数据库厂商去实现这个接口, 即驱动

具体JDBC语法书写如下:

```java
public class AppForJDBCTest {

    @Test
    public void testJDBC() throws ClassNotFoundException, SQLException {
        //1, 注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");

        //2, 获取连接对象
        String url = "jdbc:mysql://localhost:3306/sql_store";
        String username = "mybatis";
        String password = "mybatis";
        Connection connection = DriverManager.getConnection(url, username, password);

        //3, 获取执行SQL的对象Statement, 执行SQL, 返回结果
        String sql = "select * from customers limit 10";
        Statement statement = connection.createStatement();
        ResultSet resultSet = statement.executeQuery(sql);

        //4, 封装结果数据
        List<User> userList = new ArrayList<>();
        while (resultSet.next()) {
            int customer_id = resultSet.getInt("customer_id");
            String first_name = resultSet.getString("first_name");
            String last_name = resultSet.getString("last_name");
            String birth_date = resultSet.getString("birth_date");
            String phone = resultSet.getString("phone");

            User user = new User(customer_id, first_name, last_name, birth_date, phone);
            userList.add(user);
        }
        userList.stream().forEach(System.out::println);

        //5, 释放资源
        statement.close();
        resultSet.close();
    }
}

```

返回结果为:

```
User{customer_id=1, first_name='Babara', last_name='MacCaffrey', birth_date='1986-03-28', phone='781-932-9754'}
User{customer_id=2, first_name='Ines', last_name='Brushfield', birth_date='1986-04-13', phone='804-427-9456'}
User{customer_id=3, first_name='Freddi', last_name='Boagey', birth_date='1985-02-07', phone='719-724-7869'}
User{customer_id=4, first_name='Ambur', last_name='Roseburgh', birth_date='1974-04-14', phone='407-231-8017'}
User{customer_id=5, first_name='Clemmie', last_name='Betchley', birth_date='1973-11-07', phone='null'}
User{customer_id=6, first_name='Elka', last_name='Twiddell', birth_date='1991-09-04', phone='312-480-8498'}
User{customer_id=7, first_name='Ilene', last_name='Dowson', birth_date='1964-08-30', phone='615-641-4759'}
User{customer_id=8, first_name='Thacher', last_name='Naseby', birth_date='1993-07-17', phone='941-527-3977'}
User{customer_id=9, first_name='Romola', last_name='Rumgay', birth_date='1992-05-23', phone='559-181-3744'}
User{customer_id=10, first_name='Levy', last_name='Mynett', birth_date='1969-10-13', phone='404-246-3370'}
```

**以上信息均为随机生成的假信息**

原始JDBC书写出现的问题:

- 硬编码, 数据库的链接地址和用户名, 密码全都写死在了Java代码里
- 解析sql语句返回结果的代码太过繁琐
- 频繁调用数据库连接和释放资源, 造成资源浪费性能降低

MyBatis相比JDBC带来的好处:

- 将硬编码部分转移到了配置文件中, 实现了动态更改
- @Select(sql语句)注解实现了自动封装sql语句的功能
- spring.datasource 使用数据库连接池技术, 实现了连接对象的复用,
  从而避免了频繁创建新连接释放连接造成的资源浪费和性能问题

## 4. MyBatis数据库连接池

- 数据库连接池是个容器, 负责, 管理数据库连接(Connection)
- 允许应用程序重复使用一个现有的数据库连接, 不是重新建立一个
- 释放空闲时间超过最大空闲时间的连接, 归还到连接池, 避免没有释放连接而引起数据库连接遗漏
- 优点: 资源复用, 提升系统响应速度, 避免数据库连接遗漏

实现连接池:

标准接口: DataSource, 用第三方组织实现此接口Druid, Hikari(spring boot自带)

Druid(德鲁伊)

- 阿里巴巴开源的数据池连接项目
- 功能强大, 性能优秀, 是Java语言最好的数据库连接池之一

使用Druid实现数据库连接池:

(1)pom.xml引入Druid依赖

```xml

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.18</version>
</dependency>
```

(2)配置数据库的连接信息->[application.properties](MybatisDemo01%2Fsrc%2Fmain%2Fresources%2Fapplication.properties)
application.properties
(3)结果替换成了Druid数据库连接池

```
2023-06-04 11:04:12.309  INFO 49276 --- [ionShutdownHook] com.alibaba.druid.pool.DruidDataSource
```

## 5. lombok工具包

简化JavaBean的书写

相关注解功能:

- @Getter/@Setter 为所有的属性提供get/set方法
- @ToString 重写的toString方法
- @EqualsAndHashCOde equals和hashCode方法
- @Data 上方全部注解的整合
- NoArgsConstructor 为实体类生成无参构造方法
- @AllArgsConstructor 为实体类生成除static修饰的字段之外带有各参数的构造器方法

引入lombok依赖

```xml

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.26</version>
</dependency>
```

添加注解后的User类文件

```java

@Data //@Getter+@Setter+@ToString+@EqualsAndHashCode
@NoArgsConstructor //无参构造
@AllArgsConstructor //全参构造
public class User {
    private Integer customer_id;
    private String first_name;
    private String last_name;
    private String birth_date;
    private String phone;
}
```

书写界面及其简洁, 代码测试也毫无问题

# 二. MyBatis的基础操作

案例: 员工的增删改查功能

- [查询](#5-基础操作-查询)
    - [根据主键ID查询](#1-根据id查询)
    - [条件查询](#2-条件查询)
- [新增](#3-基础操作-新增)
- [更新](#4-基础操作-更新)
- [删除](#2-基础操作-删除)
    - [根据主键ID删除](#1-根据主键id删除)
    - [根据主键ID批量删除](#2-根据主键id批量删除)

## 1. 准备数据库

使用sql文件创建表, 下面我也会放出相关的sql语句
<details><summary>相关SQL语句请展开查看</summary><pre><code>

```sql
-- 部门管理
drop table if exists dept;
create table dept
(
    id          int unsigned primary key auto_increment comment '主键ID',
    name        varchar(10) not null unique comment '部门名称',
    create_time datetime    not null comment '创建时间',
    update_time datetime    not null comment '修改时间'
) comment '部门表';
insert into dept (id, name, create_time, update_time)
values (1, '学工部', now(), now()),
       (2, '教研部', now(), now()),
       (3, '咨询部', now(), now()),
       (4, '就业部', now(), now()),
       (5, '人事部', now(), now());

-- 员工管理
drop table if exists emp;
create table emp
(
    id          int unsigned primary key auto_increment comment 'ID',
    username    varchar(20) not null unique comment '用户名',
    password    varchar(32) default '123456' comment '密码',
    name        varchar(10) not null comment '姓名',
    gender      tinyint unsigned not null comment '性别, 说明: 1 男, 2 女',
    image       varchar(300) comment '图像',
    job         tinyint unsigned comment '职位, 说明: 1 班主任,2 讲师, 3 学工主管, 4 教研主管, 5 咨询师',
    entrydate   date comment '入职时间',
    dept_id     int unsigned comment '部门ID',
    create_time datetime    not null comment '创建时间',
    update_time datetime    not null comment '修改时间'
) comment '员工表';

INSERT INTO emp
(id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time)
VALUES (1, 'jinyong', '123456', '金庸', 1, '1.jpg', 4, '2000-01-01', 2, now(), now()),
       (2, 'zhangwuji', '123456', '张无忌', 1, '2.jpg', 2, '2015-01-01', 2, now(), now()),
       (3, 'yangxiao', '123456', '杨逍', 1, '3.jpg', 2, '2008-05-01', 2, now(), now()),
       (4, 'weiyixiao', '123456', '韦一笑', 1, '4.jpg', 2, '2007-01-01', 2, now(), now()),
       (5, 'changyuchun', '123456', '常遇春', 1, '5.jpg', 2, '2012-12-05', 2, now(), now()),
       (6, 'xiaozhao', '123456', '小昭', 2, '6.jpg', 3, '2013-09-05', 1, now(), now()),
       (7, 'jixiaofu', '123456', '纪晓芙', 2, '7.jpg', 1, '2005-08-01', 1, now(), now()),
       (8, 'zhouzhiruo', '123456', '周芷若', 2, '8.jpg', 1, '2014-11-09', 1, now(), now()),
       (9, 'dingminjun', '123456', '丁敏君', 2, '9.jpg', 1, '2011-03-11', 1, now(), now()),
       (10, 'zhaomin', '123456', '赵敏', 2, '10.jpg', 1, '2013-09-05', 1, now(), now()),
       (11, 'luzhangke', '123456', '鹿杖客', 1, '11.jpg', 5, '2007-02-01', 3, now(), now()),
       (12, 'hebiweng', '123456', '鹤笔翁', 1, '12.jpg', 5, '2008-08-18', 3, now(), now()),
       (13, 'fangdongbai', '123456', '方东白', 1, '13.jpg', 5, '2012-11-01', 3, now(), now()),
       (14, 'zhangsanfeng', '123456', '张三丰', 1, '14.jpg', 2, '2002-08-01', 2, now(), now()),
       (15, 'yulianzhou', '123456', '俞莲舟', 1, '15.jpg', 2, '2011-05-01', 2, now(), now()),
       (16, 'songyuanqiao', '123456', '宋远桥', 1, '16.jpg', 2, '2010-01-01', 2, now(), now()),
       (17, 'chenyouliang', '123456', '陈友谅', 1, '17.jpg', NULL, '2015-03-21', NULL, now(), now());
```

</code></pre></details>

**员工信息均为随机生成的假信息**

准备好springboot工程, 添加相关依赖, 详情请看上方总结

创建对应实体类Emp

```java

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Emp {
    private Integer id;
    private String username;
    private String password;
    private String name;
    private Short gender;
    private String image;
    private Short job;
    private LocalDate entrydate;
    private Integer deptId;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}
```

准备Mapper接口EmpMapper

```java

@Mapper
public interface EmpMapper {
    @Select("sql语句")
    List<Emp> list();
}

```

针对MyBatis封装对象时, 实体类变量名和sql数据库字段名不同导致获取不到信息的解决方案

- 方案一: 给字段起别名, 让别名与实体类型一致

```java
public interface Mapper {
    @Select("select id, username, password, name, gender, image, job, entrydate, dept_id as deptId, create_time as createTime, update_time as updateTime " +
            "from emp")
    List<Staff> showAll();
}
```

- 方案二: 通过@Results()和@Result()来对字段名进行映射, 映射目标为实体类型

```java
public interface Mapper {
    @Select("select * from emp")
    @Results(id = "reStaff", value = {
            @Result(column = "dept_id", property = "deptId"),
            @Result(column = "create_time", property = "createTime"),
            @Result(column = "update_time", property = "updateTime")
    })
    List<Staff> showAll();

    @Select("select * from emp where id=10001")
    @ResultMap("reStaff")
        //@ResultMap引入来复用上方的代码
    Staff selectById(int id);
}
```

前两种方案都很臃肿, 不太推荐使用

- 方案三: 开启驼峰命名自动映射开关 a_column自动分装成aColumn,
  需要在application.properties中开启, 原来的代码怎么写就怎么写

```properties
# camel name type
mybatis.configuration.map-underscore-to-camel-case=true
```

## 2. 基础操作-删除

### (1). 根据主键ID删除

mysql语句为:

```mysql
delete
from emp
where id = #{id};
```

Mapper接口里的删除方法如下

```java

@Mapper
public interface EmpMapper {
    @Delete("delete from emp where id = #{id}")
    void delete(Integer id);
}
```

最后通过测试来调用方法测试

```java

@SpringBootTest
public class AppForMyBatisTest {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testDelete() {
        empMapper.delete(17);
    }
}
```

结果id为17的信息被删除了, 也可以设置int返回值来查看删除了几条数据

查看mybatis调用sql语句的日志:

首先可以在application.properties中, 打开mybatis的日志, 并指定输出到控制台

```properties
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

控制台log打印:

```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@30bbcf91] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@3b97907c] will not be managed by Spring
==>  Preparing: delete from emp where id = ?
==> Parameters: 17(Integer)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@30bbcf91]
```

预编译sql语句, 使用#占位符来暂时替代数值(好处):

- 性能更高(sql语法解析检查, 优化sql, 编译sql, mybatis对这一过程进行了缓存, 有同样的sql语句可以直接拿来用,
  而占位符大大同化了条件是不同id的sql语句为同一sql语句, 预编译好后运行时只需要把传进来的数值替换占位符即可,
  大大提高了sql查询编译性能)
- 更安全(防止sql注入:通过输入的数据来修改事先定义好的SQL语句, 以达到执行代码对服务器进行攻击的方法 )

#{}和${}区别:

- #{}: 编译时有一个占位符?, sql语句参数传递时都使用
- ${}: 直接拼接sql语句, 存在sql注入问题, 对表名,列表进行动态设置时使用

### (2). 根据主键ID批量删除

mysql语句为:

```mysql
delete
from emp
where id = #{id1}
   or id = #{id2}
   or id = #{id3};
```

Mapper接口里的删除方法参数里只需要传递多个参数即可, 演示类似单个Id的删除, 故不在演示

```java

@Mapper
public interface EmpMapper {
  
    @Delete("delete from emp where id = #{id1} or id = #{id2} or id = #{id3}")
    void delete(Integer id1, Integer id2, Integer id3);
}
```

## 3. 基础操作-新增

mysql语句为

```mysql
insert into emp (username, name, gender, image, job, entrydate, dept_id, create_time, update_time)
VALUES ('tom', '汤姆', '1', '1.jpg', '1', '2005-01-01', '1', now(), now())
```

Mapper接口里的新增方法如下

```java

@Mapper
public interface EmpMapper {
    @Insert("insert into emp (username, name, gender, image, job, entrydate, dept_id, create_time, update_time)" +
            "    VALUES (#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
    void insert(Emp emp);
}
```

最后通过测试来调用方法测试

```java

@SpringBootTest
public class AppForMyBatisTest {
    @Autowired
    private EmpMapper empMapper;
    
    @Test
    public void testInsert() {
        Emp emp = new Emp();
        emp.setUsername("Tom");
        emp.setName("汤姆");
        emp.setImage("1.jpg");
        emp.setGender((short) 1);
        emp.setJob((short) 1);
        emp.setEntrydate(LocalDate.of(2000, 1, 1));
        emp.setCreateTime(LocalDateTime.now());
        emp.setUpdateTime(LocalDateTime.now());
        emp.setDeptId(1);

        //执行新增操作
        empMapper.insert(emp);
    }
}
```

测试结果插入成功:

```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@57e5396b] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@61608e1a] will not be managed by Spring
==>  Preparing: insert into emp (username, name, gender, image, job, entrydate, dept_id, create_time, update_time) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
==> Parameters: Tom(String), 汤姆(String), 1(Short), 1.jpg(String), 1(Short), 2000-01-01(LocalDate), 1(Integer), 2023-06-04T16:21:27.813802800(LocalDateTime), 2023-06-04T16:21:27.813802800(LocalDateTime)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@57e5396b]
```

新增主键返回:

- 在数据添加成功后, 需要获取插入数据库数据的主键, 如:添加套餐数据时, 还需要维护套餐菜品的关系表数据
- 实现: 在新增方法上再加上一个返回主键的注解, 将新增的id封装给emp对象 **@Options(keyProperty = "id", useGeneratedKeys =
  true)**

```java

@Mapper
public interface EmpMapper {
    @Options(keyProperty = "id", useGeneratedKeys = true)
    @Insert("insert into emp (username, name, gender, image, job, entrydate, dept_id, create_time, update_time)" +
            "    VALUES (#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
    void insert(Emp emp);
}
```

## 4. 基础操作-更新

### 根据主键id更新

mysql语句为

```mysql
update emp
set username    = #{username},
    name        = #{name},
    gender      = #{gender},
    image       = #{image},
    job         = #{job},
    entrydate   = #{entrydate},
    dept_id     = #{deptId},
    update_time = #{updateTime}
where id = #{id}
```

Mapper接口里的更新方法如下

```java

@Mapper
public interface EmpMapper {
    @Update("update emp set username = #{username}, name = #{name}, gender = #{gender}, image = #{image}, job = #{job}, entrydate = #{entrydate}, dept_id = #{deptId}, update_time = #{updateTime}" +
            "    where id = #{id}")
    void update(Emp emp);
}
```

最后通过测试来调用方法测试

```java

@SpringBootTest
public class AppForMyBatisTest {
    @Autowired
    private EmpMapper empMapper;
    
    @Test
    public void testUpdate() {
        Emp emp = new Emp();
        emp.setId(19);
        emp.setUsername("Tom2");
        emp.setName("汤姆2");
        emp.setImage("1.jpg");
        emp.setGender((short) 1);
        emp.setJob((short) 1);
        emp.setEntrydate(LocalDate.of(2000, 1, 1));
        emp.setCreateTime(LocalDateTime.now());
        emp.setUpdateTime(LocalDateTime.now());
        emp.setDeptId(1);

        empMapper.update(emp);
    }
}
```

测试结果更新成功:

```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6097fca9] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@421d54b3] will not be managed by Spring
==>  Preparing: update emp set username = ?, name = ?, gender = ?, image = ?, job = ?, entrydate = ?, dept_id = ?, update_time = ? where id = ?
==> Parameters: Tom2(String), 汤姆2(String), 1(Short), 1.jpg(String), 1(Short), 2000-01-01(LocalDate), 1(Integer), 2023-06-04T16:53:03.568252300(LocalDateTime), 19(Integer)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6097fca9]
```

## 5. 基础操作-查询

### (1). 根据id查询

mysql语句为

```mysql
select *
from emp
where id = 19
```

Mapper接口里的查询方法如下

```java

@Mapper
public interface EmpMapper {
    @Select("select * from emp where id = #{id}")
    Emp selectById(Integer id);
}
```

最后通过测试来调用方法测试

```java

@SpringBootTest
public class AppForMyBatisTest {
    @Autowired
    private EmpMapper empMapper;
    
    @Test
    public void testSelectById() {
        Emp emp = empMapper.selectById(19);
        System.out.println(emp);
    }
}
```

测试结果查询成功:

```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@65be88ae] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@7cfb0c4c] will not be managed by Spring
==>  Preparing: select * from emp where id = ?
==> Parameters: 19(Integer)
<==    Columns: id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time
<==        Row: 19, Tom2, 123456, 汤姆2, 1, 1.jpg, 1, 2000-01-01, 1, 2023-06-04 16:31:56, 2023-06-04 16:53:04
<==      Total: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@65be88ae]
Emp(id=19, username=Tom2, password=123456, name=汤姆2, gender=1, image=1.jpg, job=1, entrydate=2000-01-01, deptId=1, createTime=2023-06-04T16:31:56, updateTime=2023-06-04T16:53:04)
```

### (2). 条件查询

mysql语句为

```mysql
select *
from emp
where name like '%${name}%'
  and gender = #{gender}
  and entrydate between #{begin} and #{end}
order by update_time desc
```

因为是模糊查询, like后边是字符串的'% %', 所以需要进行字符串的拼接, 采用${}占位符, 如果是#{}, like后边会变成'%?%',
就没法字符串里的?替换成真实值了, 但是依然存在一些性能和安全问题, 我们先实现该查询方法起, 实现过后下方会给出对应的解决方案

Mapper接口里的条件查询方法如下

```java

@Mapper
public interface EmpMapper {
    //条件查询
    @Select("select * from emp where name like '%${name}%' and gender = #{gender} and entrydate between #{begin} and #{end}" +
            "    order by update_time desc")
    List<Emp> conditionalQuery(String name, Short gender, LocalDate begin, LocalDate end);
}
```

最后通过测试来调用方法测试

```java

@SpringBootTest
public class AppForMyBatisTest {
    @Autowired
    private EmpMapper empMapper;
  
    @Test
    public void testConditionalQuery() {
        List<Emp> empList = empMapper.conditionalQuery("张", (short) 1, LocalDate.of(2010, 1, 1), LocalDate.of(2020, 1, 1));
        empList.stream().forEach(System.out::println);
    }
}
```

测试结果查询成功:

```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@1e98b788] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@616a06e3] will not be managed by Spring
==>  Preparing: select * from emp where name like '%张%' and gender = ? and entrydate between ? and ? order by update_time desc
==> Parameters: 1(Short), 2010-01-01(LocalDate), 2020-01-01(LocalDate)
<==    Columns: id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time
<==        Row: 2, zhangwuji, 123456, 张无忌, 1, 2.jpg, 2, 2015-01-01, 2, 2023-06-04 15:28:00, 2023-06-04 15:28:00
<==      Total: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@1e98b788]
Emp(id=2, username=zhangwuji, password=123456, name=张无忌, gender=1, image=2.jpg, job=2, entrydate=2015-01-01, deptId=2, createTime=2023-06-04T15:28, updateTime=2023-06-04T15:28)
```
之前的模糊查询时采用了${}拼接符, 因此存在性能和安全问题, 解决方案是采用数据库的字符串拼接函数 concat
```mysql
select *
from emp
where name like concat('%', #{name}, '%')
  and gender = #{gender}
  and entrydate between #{begin} and #{end}
order by update_time desc
```
```java
@Mapper
public interface EmpMapper {
    //条件查询
    @Select("select * from emp where name like concat('%', #{name}, '%') and " +
            "gender = #{gender} and " +
            "entrydate between #{begin} and #{end}" +
            "    order by update_time desc")
    List<Emp> conditionalQuery(String name, Short gender, LocalDate begin, LocalDate end);
}
```
完美解决安全和性能的问题, 让我们继续再次测试下

结果依然一样可以查询到:
```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@46c10083] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@24dd44f9] will not be managed by Spring
==>  Preparing: select * from emp where name like concat('%', ?, '%') and gender = ? and entrydate between ? and ? order by update_time desc
==> Parameters: 张(String), 1(Short), 2010-01-01(LocalDate), 2020-01-01(LocalDate)
<==    Columns: id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time
<==        Row: 2, zhangwuji, 123456, 张无忌, 1, 2.jpg, 2, 2015-01-01, 2, 2023-06-04 15:28:00, 2023-06-04 15:28:00
<==      Total: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@46c10083]
Emp(id=2, username=zhangwuji, password=123456, name=张无忌, gender=1, image=2.jpg, job=2, entrydate=2015-01-01, deptId=2, createTime=2023-06-04T15:28, updateTime=2023-06-04T15:28)
```
推荐使用该方法来实现类似模糊查询的诸多查询
# 三. MyBatis的XML映射文件配置SQL语句
- XML映射文件的名称与Mapper接口一致, 并且将XML映射文件和Mapper接口放在相同包下(同包同名, 一个接口对应一个配置文件)
- XML映射文件的namespace属性与Mapper接口全限定名(com.itstudy.mapper.Emp)一致
- XML映射文件中sql语句的id与Mapper接口中的方法名一致, 并保持返回类型一致

前期配置依然需要spring boot项目和配置好相关的配置文件application.properties, pom.xml
## 1. 配置XML文件
首先新建好一个Mapper接口
```java
@Mapper
public interface EmpMapper {

    List<Emp> list(String name, Short gender, LocalDate begin, LocalDate end);

}
```
在resources下建包, 包名不能用.分隔, 得用/分隔, com/itstudy/mapper

再右键新建 EmpMapper.xml

配置相关约束和sql语句编写
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itstudy.mapper.EmpMapper">
    <!--查询标签<select>, id需要与Mapper接口里的方法名一致, 
    resultType为单条记录所对应的全限定名类型即com.itstudy.pojo.Emp, -->
    <select id="conditionalQuery" resultType="com.itstudy.pojo.Emp">
        select *
        from emp
        where name like concat('%', #{name}, '%')
          and gender = #{gender}
          and entrydate between #{begin} and #{end}
        order by update_time desc
    </select>
</mapper>
```
最后通过调用方法进行测试

```java

@SpringBootTest
public class AppForMyBatisTest {
    @Autowired
    private EmpMapper empMapper;
    
    @Test
    public void testConditionalQuery() {
        List<Emp> empList = empMapper.conditionalQuery("张", (short) 1, LocalDate.of(2010, 1, 1), LocalDate.of(2020, 1, 1));
        empList.stream().forEach(System.out::println);
    }
}
```
测试结果查询成功:
```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@51ff3c4b] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@6d0290d8] will not be managed by Spring
==>  Preparing: select * from emp where name like concat('%', ?, '%') and gender = ? and entrydate between ? and ? order by update_time desc
==> Parameters: 张(String), 1(Short), 2010-01-01(LocalDate), 2020-01-01(LocalDate)
<==    Columns: id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time
<==        Row: 2, zhangwuji, 123456, 张无忌, 1, 2.jpg, 2, 2015-01-01, 2, 2023-06-04 15:28:00, 2023-06-04 15:28:00
<==      Total: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@51ff3c4b]
Emp(id=2, username=zhangwuji, password=123456, name=张无忌, gender=1, image=2.jpg, job=2, entrydate=2015-01-01, deptId=2, createTime=2023-06-04T15:28, updateTime=2023-06-04T15:28)
```
开发相关插件(MyBatisX)下载, 请自行下载, 该插件能在xml里的sql语句和Mapper接口对应的方法来进行快速切换

使用MyBatis注解主要是增删改查等一些简单操作,
在对应的复杂工程的SQL语句查询的环境下, 建议使用XML文件进行开发

## 2. 动态SQL语句
随着用户的输入或外部条件的变化而变化的SQL语句, 如果用户有一个条件没有写, 原来的SQL语句就没法执行了, 所以需要动态SQL语句来执行
### (1). 动态SQL-if & where
只需要在EmpMapper.xml文件里sql语句的书写加上`<if type="条件">`标签, where替换成`<where></where>`标签,
`<where>`标签能够自动把sql语句里把不满足条件的语句自动从sql语句中剔除, 也能自动去除多余的and或者or, 都不成立就不会生成where子句
```xml

<select id="conditionalQuery" resultType="com.itstudy.pojo.Emp">
    select *
    from emp
    <where>
        <if test="name != null">
            name like concat('%', #{name}, '%')
        </if>
        <if test="gender != null">
            and gender = #{gender}
        </if>
        <if test="begin != null and end != null">
            and entrydate between #{begin} and #{end}
        </if>
    </where>
    order by update_time desc
</select>
```
测试姓名不写的情况下是否能查询
```java
@SpringBootTest
class MybatisDemo02ApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testConditionalQuery() {
        List<Emp> empList = empMapper.conditionalQuery(null, (short) 1, LocalDate.of(2010, 1, 1), LocalDate.of(2020, 1, 1));
        empList.stream().forEach(System.out::println);
    }

}
```
运行结果显示可以查询其他符合条件的
```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@4962b41e] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@4a50c746] will not be managed by Spring
==>  Preparing: select * from emp WHERE gender = ? and entrydate between ? and ? order by update_time desc
==> Parameters: 1(Short), 2010-01-01(LocalDate), 2020-01-01(LocalDate)
<==    Columns: id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time
<==        Row: 2, zhangwuji, 123456, 张无忌, 1, 2.jpg, 2, 2015-01-01, 2, 2023-06-04 15:28:00, 2023-06-04 15:28:00
<==        Row: 5, changyuchun, 123456, 常遇春, 1, 5.jpg, 2, 2012-12-05, 2, 2023-06-04 15:28:00, 2023-06-04 15:28:00
<==        Row: 13, fangdongbai, 123456, 方东白, 1, 13.jpg, 5, 2012-11-01, 3, 2023-06-04 15:28:00, 2023-06-04 15:28:00
<==        Row: 15, yulianzhou, 123456, 俞莲舟, 1, 15.jpg, 2, 2011-05-01, 2, 2023-06-04 15:28:00, 2023-06-04 15:28:00
<==        Row: 16, songyuanqiao, 123456, 宋远桥, 1, 16.jpg, 2, 2010-01-01, 2, 2023-06-04 15:28:00, 2023-06-04 15:28:00
<==      Total: 5
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@4962b41e]
Emp(id=2, username=zhangwuji, password=123456, name=张无忌, gender=1, image=2.jpg, job=2, entrydate=2015-01-01, deptId=2, createTime=2023-06-04T15:28, updateTime=2023-06-04T15:28)
Emp(id=5, username=changyuchun, password=123456, name=常遇春, gender=1, image=5.jpg, job=2, entrydate=2012-12-05, deptId=2, createTime=2023-06-04T15:28, updateTime=2023-06-04T15:28)
Emp(id=13, username=fangdongbai, password=123456, name=方东白, gender=1, image=13.jpg, job=5, entrydate=2012-11-01, deptId=3, createTime=2023-06-04T15:28, updateTime=2023-06-04T15:28)
Emp(id=15, username=yulianzhou, password=123456, name=俞莲舟, gender=1, image=15.jpg, job=2, entrydate=2011-05-01, deptId=2, createTime=2023-06-04T15:28, updateTime=2023-06-04T15:28)
Emp(id=16, username=songyuanqiao, password=123456, name=宋远桥, gender=1, image=16.jpg, job=2, entrydate=2010-01-01, deptId=2, createTime=2023-06-04T15:28, updateTime=2023-06-04T15:28)

```

#### 相关案例, 动态更新员工信息
Mapper接口方法编写
```java
@Mapper
public interface EmpMapper {
    void update(Emp emp);
}
```
EmpMapper.xml配置文件编写
```xml

<update id="update">
    update emp
    <set>
        <if test="username != null">
            username = #{username},
        </if>
        <if test="name != null">
            name = #{name},
        </if>
        <if test="gender != null">
            gender = #{gender},
        </if>
        <if test="image != null">
            image = #{image},
        </if>
        <if test="entrydate != null">
            entrydate = #{entrydate},
        </if>
        <if test="deptId != null">
            dept_id = #{deptId},
        </if>
        <if test="updateTime != null">
            update_time = #{updateTime}
        </if>
    </set>
    where id = #{id}
</update>
```
测试类进行运行, 只更新几个字段的内容
```java
@SpringBootTest
class MybatisDemo02ApplicationTests {
    @Autowired
    private EmpMapper empMapper;
    
    @Test
    public void testUpdate() {
        Emp emp = new Emp();
        emp.setId(19);
        emp.setUsername("tom111");
        emp.setImage("2.jpg");
        emp.setUpdateTime(LocalDateTime.now());

        empMapper.update(emp);
    }
}
```
运行结果更新成功:

```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@475f5672] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@251c8145] will not be managed by Spring
==>  Preparing: update emp set username = ?, image = ?, update_time = ? where id = ?
==> Parameters: tom111(String), 2.jpg(String), 2023-06-04T20:24:38.677394200(LocalDateTime), 19(Integer)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@475f5672]

```
表中其他没有更新的字段依然存在, 不会更新成null

19,tom111,123456,汤姆2,1,2.jpg,1,2000-01-01,1,2023-06-04 16:31:56,2023-06-04 20:24:39

更新时也会像and或or, 存在逗号多出来的情况, 跟where标签一样请使用set标签`<set></set>`, 这个标签可以删除多余的逗号

### (2). 动态SQL-foreach
员工数据的批量删除, 删除id为1, 2, 3的员工信息
```mysql
delete from emp where id in (1, 2, 3)
```
准备Mapper里的批量删除的方法, 把id的范围以集合的方式传进去
```java
@Mapper
public interface EmpMapper {
    void deleteByIds(List<Integer> idList);
}
```
在EmpMapper.xml里的语句书写方式, 这时就需要用到`<foreach>`标签
```xml

<delete id="deleteByIds">
    delete
    from emp
    where id in
    <foreach collection="idList" item="id" separator="," open="(" close=")">
        #{id}
    </foreach>

</delete>
```
进行测试类编写, 传进去一个List集合idList, 含有id(1, 2, 3)
```java
@SpringBootTest
class MybatisDemo02ApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testDeleteByIds() {
        List<Integer> idList = new ArrayList<>();
        Collections.addAll(idList, 1, 2, 3);
        empMapper.deleteByIds(idList);
    }
}
```

运行的结果显示删除成功

```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@195113de] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@4a50c746] will not be managed by Spring
==>  Preparing: delete from emp where id in ( ? , ? , ? )
==> Parameters: 1(Integer), 2(Integer), 3(Integer)
<==    Updates: 3
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@195113de]
```
### (3). 动态SQL-sql & include
解决重复的SQL片段

`<sql id="commonSelect">SQL片段</sql>`这个标签包裹重复的SQL片段

`<inlude refid="commonSelect"/>`这个标签可以让SQL语句在多出进行复用

具体EmpMapper.xml配置如下:
```xml

<sql id="commonSelect">
    select *
    from emp
</sql>

<select id="conditionalQuery" resultType="com.itstudy.pojo.Emp">
<include refid="commonSelect"/>
<where>
    <if test="name != null">
        name like concat('%', #{name}, '%')
    </if>
    <if test="gender != null">
        and gender = #{gender}
    </if>
    <if test="begin != null and end != null">
        and entrydate between #{begin} and #{end}
    </if>
</where>
order by update_time desc
</select>

<select id="getById" resultType="com.itstudy.pojo.Emp">
<include refid="commonSelect"/>
where id = #{id}
</select>
```
测试结果依然可以查询到员工的相关信息(只对根据id查询方法进行了测试):
```
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@5b2b8d86] was not registered for synchronization because synchronization is not active
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@20a24edf] will not be managed by Spring
==>  Preparing: select * from emp where id = ?
==> Parameters: 1(Integer)
<==    Columns: id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time
<==        Row: 1, jinyong, 123456, 金庸, 1, 1.jpg, 4, 2000-01-01, 2, 2023-06-04 20:50:16, 2023-06-04 20:50:16
<==      Total: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@5b2b8d86]
Emp(id=1, username=jinyong, password=123456, name=金庸, gender=1, image=1.jpg, job=4, entrydate=2000-01-01, deptId=2, createTime=2023-06-04T20:50:16, updateTime=2023-06-04T20:50:16)
```