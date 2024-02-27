@[TOC]

# 一. spring整合mybatis

首先整合mybatis用于spring的坐标(pom.xml),
下面坐标缺一不可

```xml

<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.9</version>
</dependency>

<dependency>
<groupId>mysql</groupId>
<artifactId>mysql-connector-java</artifactId>
<version>8.0.28</version>
</dependency>

<dependency>
<groupId>com.alibaba</groupId>
<artifactId>druid</artifactId>
<version>1.2.18</version>
</dependency>

<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-jdbc</artifactId>
<version>5.3.27</version>
</dependency>
        <!--mybatis spring版本的,  需要与springframework版本有对应关系-->
<dependency>
<groupId>org.mybatis</groupId>
<artifactId>mybatis-spring</artifactId>
<version>2.0.7</version>
</dependency>

<dependency>
<groupId>org.projectlombok</groupId>
<artifactId>lombok</artifactId>
<version>1.18.26</version>
</dependency>
```

采用注解模式开发

首先准备好封装的实体类Account

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Account {
    public Integer id;
    public String username;
    public String password;
    public String name;
    public Short gender;
    public String image;
    public Short job;
    public LocalDate entrydate;
    public Integer deptId;
    public LocalDateTime createTime;
    public LocalDateTime updateTime;

}
```

编写要查询sql语句的Mapper接口AccountDao

```java
/*
* 主要写查询语句
* */
@Mapper
public interface AccountDao {

    @Delete("delete from emp where id = #{id}")
    void delete(Integer id);

    @Results(id = "reStaff", value = {
            @Result(column = "dept_id", property = "deptId"),
            @Result(column = "create_time", property = "createTime"),
            @Result(column = "update_time", property = "updateTime")
    })
    @Select("select * from emp")
    List<Account> findAll();

    @Select("select * from emp where id = #{id}")
    @ResultMap("reStaff")
    Account findById(Integer id);

    @Insert("insert into emp(username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time) " +
            "VALUES (#{username}, #{password}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
    @ResultMap("reStaff")
    void insert(Account account);

}
```

现在开始准备数据库连接池即JDBC的连接对象配置---JdbcConfig, 数据库的一些属性从外部jdbc.properties导入, 在主配置文件中导入

```java
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;


    //1, 定义一个方法获得要管理的bean
    //2, 添加@Bean表示返回的是一个Bean
    @Bean
    public DataSource dataSource() {
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(username);
        ds.setPassword(password);
        return ds;
    }
}
```

数据库对象准备好后, 开始配备MyBatisConfig, 连接数据库生成数据库连接池, 同时进行需要查询的sql语句Mapper接口映射

```java
public class MyBatisConfig {

    /*
    * 专门获取SqlSessionFactory的bean的方法
     * dataSource即为JdbcConfig里配置的DataSource bean
     * setTypeAliasesPackage指定你sql语句查询时, 复杂类型里的成员变量是否可以单独拿出来用如#{name}, 不需要点语法才能用#{ACcount.name}
    * */
    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource) {
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        ssfb.setTypeAliasesPackage("com.itstudy.domain");
        ssfb.setDataSource(dataSource);
        return ssfb;
    }

    /*
    * Mapper映射配置, 配置自己要查询的sql语句
    * */
    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
        msc.setBasePackage("com.itstudy.dao");
        return msc;
    }

}
```

最后准备基础的配置文件SpringConfig, 引入bean所在的包,
mybatis用于jdbc连接的配置文件[jdbc.properties](SpringDemo13_mybatis%2Fsrc%2Fmain%2Fresources%2Fjdbc.properties),
导入JdbcConfig和MyBatisConfig

```java
@Configuration
@ComponentScan("com.itstudy")
@PropertySource({"classpath:jdbc.properties"})
@Import({JdbcConfig.class, MyBatisConfig.class})
public class SpringConfig {

}
```

设置服务层对象AccountServiceImpl.java, 自动装配注入Mapper依赖来执行sql语句

```java
@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    @Override
    public void insert(Account account) {
        accountDao.insert(account);
    }

    @Override
    public void delete(Integer id) {
        accountDao.delete(id);
    }

    @Override
    public Account findById(Integer id) {
        return accountDao.findById(id);
    }

    @Override
    public List<Account> findAll() {
        return accountDao.findAll();
    }
}
```

最后在测试类AppTestForMybatis.java里调用服务层的bean来执行服务层里的方法, 只进行了插入单条数据和获取全部数据的方法

```java
public class AppTestForMybatis {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);

        AccountService accountService = ctx.getBean(AccountService.class);

        Account account1 = new Account();
        account1.setUsername("tom2");
        account1.setName("汤姆2");
        account1.setImage("2.jpg");
        account1.setGender((short) 1);
        account1.setJob((short) 2);
        account1.setEntrydate(LocalDate.of(2010, 1, 1));
        account1.setCreateTime(LocalDateTime.now());
        account1.setUpdateTime(LocalDateTime.now());

        accountService.insert(account1);

        List<Account> accountList = accountService.findAll();
        accountList.stream().forEach(System.out::println);

    }
}

```

测试结果如下:

```
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
6月 05, 2023 3:52:55 下午 com.alibaba.druid.support.logging.JakartaCommonsLoggingImpl info
信息: {dataSource-1} inited
Account(id=1, username=jinyong, password=123456, name=金庸, gender=1, image=1.jpg, job=4, entrydate=2000-01-01, deptId=2, createTime=2023-06-04T20:50:16, updateTime=2023-06-04T20:50:16)
Account(id=2, username=zhangwuji, password=123456, name=张无忌, gender=1, image=2.jpg, job=2, entrydate=2015-01-01, deptId=2, createTime=2023-06-04T20:50:16, updateTime=2023-06-04T20:50:16)
Account(id=3, username=yangxiao, password=123456, name=杨逍, gender=1, image=3.jpg, job=2, entrydate=2008-05-01, deptId=2, createTime=2023-06-04T20:50:16, updateTime=2023-06-04T20:50:16)
........
Account(id=17, username=chenyouliang, password=123456, name=陈友谅, gender=1, image=17.jpg, job=null, entrydate=2015-03-21, deptId=null, createTime=2023-06-04T20:50:16, updateTime=2023-06-04T20:50:16)
Account(id=18, username=Tom, password=123456, name=汤姆, gender=1, image=1.jpg, job=1, entrydate=2000-01-01, deptId=1, createTime=2023-06-04T20:51:06, updateTime=2023-06-04T20:51:06)
Account(id=21, username=tom2, password=null, name=汤姆2, gender=1, image=2.jpg, job=2, entrydate=2010-01-01, deptId=null, createTime=2023-06-05T15:52:55, updateTime=2023-06-05T15:52:55)
```

结果显示插入成功, 返回全部数据也没有问题
___

# 二. spring整合junit

第一步依然导包

```xml

<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-test</artifactId>
  <version>5.3.27</version>
</dependency>
```

在测试类上注解上spring测试类运行器@RunWith(SpringJUnit4ClassRunner.class)

注解上运行的环境即SpringConfig配置文件@ContextConfiguration(classes = SpringConfig.class)

在需要测试的方法上打上@Test, 即可单独测试该方法

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfig.class)
public class AppTestByJunit {
    @Autowired
    private AccountService accountService;

    @Test
    public void testFindById() {
        System.out.println(accountService.findById(18));
    }
}
```

测试结果为返回一条员工数据

```
6月 05, 2023 4:23:58 下午 com.alibaba.druid.support.logging.JakartaCommonsLoggingImpl info
信息: {dataSource-1} inited
Account(id=18, username=Tom, password=123456, name=汤姆, gender=1, image=1.jpg, job=1, entrydate=2000-01-01, deptId=1, createTime=2023-06-04T20:51:06, updateTime=2023-06-04T20:51:06)
```

类运行器和上下文配置类在以后开发中几乎不会变.