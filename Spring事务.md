@[TOC]

# 一. Spring事务简介
- 事务作用: 在数据层保障一系列的数据库操作同成功同失败
- Spring事务作用: 在数据层或业务保障一些列的数据库操作同成功同失败
# 二. 案例: 银行账户转账
- 需求: 实现任意两个账户间的转账操作
- 需求微缩: A账户减钱, B账户加钱

| id | name  | money |
|----|-------|-------|
| 1  | Tom   | 1000  |
| 2  | Jerry | 1000  |
程序正常执行时, 账户金额A减B加, 没有问题

程序出现异常后, 转账失败, 但是异常之前操作成功, 异常之后操作失败, 整体业务失败

(1). 数据层提供基础操作: 指定账户减钱(outMoney), 指定账户加钱(inMoney)
```java
@Mapper
public interface AccountDao {
    @Update("update account set account_money = account_money + #{money} where account_name = #{name}")
    void inMoney(@Param("name")String name, @Param("money") Double money);

    @Update("update account set account_money = account_money - #{money} where account_name = #{name}")
    void outMoney(@Param("name")String name, @Param("money") Double money);

}
```
(2). 业务层提供转账操作(transfer): 调用减钱与加钱的操作
```java
public interface AccountService {
  /*
   * 转账操作
   * @param out 传出方
   * @param in 转入方
   * @param money 金额
   * */
  @Transactional
  void transfer(String out, String in, Double money);
}
@Service
public class AccountServiceImpl implements AccountService {
    @Autowired
    private AccountDao accountDao;

    @Override
    public void transfer(String out, String in, Double money) {
        accountDao.outMoney(out, money);
        //int i = 1/0; //开启异常
        accountDao.inMoney(in, money);
    }
}
```
(3). 提供两个账号和操作金额执行转账操作
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfig.class)
public class AccountServiceTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void testTransfer() {
        accountService.transfer("Tom", "Jerry", 100.0);
    }
}
```
(4). 基于[Spring整合MyBatis](#1-spring整合mybatis)环境搭建上述操作
## 1. 开启事务之添加Spring事务管理
一般添加事务不直接添加在实现类的方法上, 而是直接添加在实现类的接口上,
在需要添加事务的方法所对应的接口方法上打上注解`@Transactional`, 好处是降低耦合,
写在接口上就会为接口里的所有方法开启事务
```java
public interface AccountService {
    /*
    * 对指定方法的接口方法添加事务@Transactional
    *
    * */
    @Transactional
    void transfer(String out, String in, Double money);
}
```
## 2. 开启事务之设置事务管理器
因为是JDBC的事务, 所以需要在JDBC的配置文件中设置事务管理器, 同时还需要注入DataSource
```java
public class JdbcConfig {
/*
    * 定义事务管理器
    * 添加数据库资源
    * */
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}
```
## 3. 开启事务之设注解事务驱动`@EnableTransactionManagement`
```java
@Configuration
@ComponentScan("com.itstudy")
@PropertySource({"classpath:jdbc.properties"})
@Import({JdbcConfig.class, MybatisConfig.class})
@EnableTransactionManagement
public class SpringConfig {
}
```
# 三. Spring事务原理 --- 事务角色
## 1. 事务管理员
- 发起事务方, 在Spring中通常指代业务层开启事务的方法
Spring开启第三个事务, 让另外两个执行事务的操作全部加入这个第三方事务
## 2. 事务协调员
- 加入事务方, 在Spring中通常指代数据层方法, 也可以是业务层方法
即为AccountDao里的两条Sql语句的执行方法, 这两个方法, 分别有自己对应的事务, 所以当一个事务有异常时另一个事务并不会回滚操作,
当这两个事务都加入第三方同一个事务时 ,有一个除了异常, 另外一个方法也可以回滚了
```java
@Mapper
public interface AccountDao {
    //事务方1
    @Update("update account set account_money = account_money + #{money} where account_name = #{name}")
    void inMoney(@Param("name")String name, @Param("money") Double money);

    //事务方2
    @Update("update account set account_money = account_money - #{money} where account_name = #{name}")
    void outMoney(@Param("name")String name, @Param("money") Double money);

}
```

# 四. Spring事务属性
## 1. 事务配置

![](事务属性.PNG)

[图片来自B站黑马程序员---Spring事务属性](https://www.bilibili.com/video/BV1Fi4y1S7ix/?p=42&spm_id_from=pageDriver&vd_source=b1a441fcb369fd950d8bf49580ca3248)

均可以采用注解的形式来开启
## 2. 案例: 转账业务追加日志
- 基于转账操作案例添加日志模块, 实现数据库中记录日志
- 业务层转账操作(transfer), 调用减钱, 加钱与记录日志功能
实现效果预期: 无论转账操作是否成功, 均进行转账操作的日志留痕

准备好记录日志的表, 和实现方法

| id | account_info | create_date |
|----|-----------|------------|
| 1  | 转账操作有Tom到Jerry,金额:100.0  | 2023-06-07 21:05:49    |

Dao层下执行sql语句的接口方法
```java
@Mapper
public interface LogDao {

    @Insert("insert into account_logs(account_info, crate_date) values (#{info}, now())")
    void log(String info);
}
```
新建一个日志业务层调用该接口方法, 并开启事务
```java
public interface LogService {
  @Transactional
  void log(String out, String in, Double money);
}
@Service
public class LogServiceImpl implements LogService {
    @Autowired
    private LogDao logDao;
    @Override
    public void log(String out, String in, Double money) {
        logDao.log("转账操作由" + out + "到" + in + ",金额:" + money);
    }
}

```
在执行操作的业务层调用LogService日志业务接口, 并自动注入接口LogService的实现类, 为保证记入日志的方法一定执行,
采取try...finally的方式来执行
```java
@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    @Autowired
    private LogService logService;

    @Override
    public void transfer(String out, String in, Double money) {
        try {
            accountDao.outMoney(out, money);
            //int i = 1/0; //开启异常
            accountDao.inMoney(in, money);
        } finally {
            logService.log(out, in, money);
        }
    }
}
```
运行测试程序:
![](log_add.PNG)
日志添加成功

但是开启异常后开启运行测试程序发现日志记录失败, 因为同一个事务内同成功同失败, 现在加以改进实现日志必须保留,
请转看下一条目, 事务传播行为

## 3. 事务传播行为
事务协调员对事物管理员所携带事务的处理态度

![](事务传播行为.PNG)

[图片来自B站黑马程序员---Spring事务属性](https://www.bilibili.com/video/BV1Fi4y1S7ix/?p=42&spm_id_from=pageDriver&vd_source=b1a441fcb369fd950d8bf49580ca3248)

为解决上述日志记录失败问题, 需要记录日志的事务不执行事务管理员所携带的事务,在日志接口上注解`@Transactionalho`加上条件变成`@Transactional(propagation = Propagation.REQUIRES_NEW)`,
让日志不参加到同一个事务中, 自己新开一个事务
```java
public interface LogService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    void log(String out, String in, Double money);
}
```
再来开启异常后开启运行测试程序
```java
@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    @Autowired
    private LogService logService;

    @Override
    public void transfer(String out, String in, Double money) {
        try {
            accountDao.outMoney(out, money);
            int i = 1/0; //开启异常
            accountDao.inMoney(in, money);
        } finally {
            logService.log(out, in, money);
        }
    }
}
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfig.class)
public class AccountServiceTest {

  @Autowired
  private AccountService accountService;

  @Test
  public void testTransfer() {
    accountService.transfer("Tom", "Jerry", 150.0);
  }
}

```
账户金额表:

![](account01.PNG)

没有发生变动

日志记录表:

![](log01.PNG)

成功记录一条转账日志