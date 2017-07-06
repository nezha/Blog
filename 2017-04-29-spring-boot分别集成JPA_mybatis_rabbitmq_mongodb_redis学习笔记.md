---
title: "spring-boot分别集成JPA mybatis rabbitmq mongodb redis学习笔记"
date: 2017-04-29 00:00:00
category: Spring Boot
tags: [Spring Boot,Spring Jpa,mybatis,rabbitmq,mongodb,redis]
---

## Spring Boot学习笔记--全注解方式

> [Spring Boot教程与Spring Cloud教程](http://git.oschina.net/didispace/SpringBoot-Learning)

# # pring boot中文帮助文档

[Spring Boot Reference Guide中文翻译 -《Spring Boot参考指南》](https://qbgbook.gitbooks.io/spring-boot-reference-guide-zh/content/)---说明：本文档翻译的版本：1.4.1.RELEASE。

## 常用注解

- `@RestController`
返回json形式的结果，便于前后端分离。是@ResponseBody 和 @Controller的组合体
- `@RequestMapping`
配置url映射
- `@EnableAutoConfiguration`
- `@Configuration`
- `@ComponentScan`
- `@SpringBootApplication`

很多Spring Boot开发者总是使用 `@Configuration` ， `@EnableAutoConfiguration` 和 `@ComponentScan` 注解他们的main类。由于这些注解被如此频繁地一块使用（特别是你遵循以上最佳实践时），Spring Boot提供一个方便的 `@SpringBootApplication` 选择。

该 `@SpringBootApplication` 注解等价于以默认属性使用 `@Configuration` ， `@EnableAutoConfiguration` 和 `@ComponentScan` 。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

- `@ConfigurationProperties`

属性注入,prefix指代注入的配置来自connection的对象

```java
@Component
@ConfigurationProperties(prefix="connection")
public class ConnectionSettings {
    private String username;
    private InetAddress remoteAddress;
    // ... getters and setters
}
```

```xml
connection:
  name: 赵武
  age: 18
  job: Java研发工程师
```

为了使用`@ConfigurationProperties` beans，你可以使用与其他任何bean相同的方式注入它们

``` java
@Service
public class MyService {
    @Autowired
    private ConnectionSettings connection;
    //...
    @PostConstruct
    public void openConnection() {
        Server server = new Server();
        this.connection.configure(server);
    }
}
```

- `@EnableConfigurationProperties`
- `@Component`和`@Bean`
`@Bean`主要被用在方法上，来显式声明要用生成的类
- `@Profiles`

Spring Profiles提供了一种隔离应用程序配置的方式，并让这些配置只能在特定的环境下生效。

```xml
spring:
  profiles:
    active: dev
```

- `@Value`

用于获取配置文件下的配置项

```xml
people:
  name: 赵武
  age: 18
  job: Java研发工程师
```

```java
    @Value("${people.name}")
    private String name;
```

- `@Controller`

> `@PathVariable`,`@RequestParam`
> `@GetMapping`,`@PostMapping`:Get 或Post方式的请求，组合模式

`@PathVariable`的使用，获取请求参数

```java
@RequestMapping(value="/hello/{id}",method = RequestMethod.GET)
public String sayhello(@PathVariable("id")Integer myid){
    return "id:"+myid;
}
```

`@RequestParam`的使用，获取传统方式的参数

```java
@RequestMapping(value="/hi",method = RequestMethod.GET)
public String sayhi(@RequestParam(value = "id",required = false,defaultValue = "100")Integer myid){
    return "id:"+myid;
}
```

## spring data JPA  -- 单数据源

> 具体的实现代码demo：[Spring-Boot-Restful-JPA的demo程序](https://github.com/nezha/Spring-Boot-Restful-JPA)
> 
> 定义了对象持久化的标准，主要是对Hibernate的整合
> 
> 现阶段发现和mybatis的直观操作很一致，都是可能集中管理数据库连接与释放，JPA想比较于mybatis可以自动建表，不知道算不算一种优势，在我看来不算是。毕竟表结构基本上数据库工程师都搞定了的事。

1.加入依赖

```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

2.配置数据库和JPA

> ddl-auto:创建的方式

```xml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/coder
    username: root
    password: root
  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true
```

3.创建Mapper对象

> @Entity: 持久化实例
> 
> @Table: 自定义表的名称

```java
@Entity
//@Table(name = "programmer")
public class Coder {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;
    private String name;
    private Integer age;
    public Coder(){
        
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

4.集成JpaRepository

> 主要是基于Hibernate的

```java
public interface CoderRepository extends JpaRepository<Coder,Integer> {
    //这个是扩展开来的查询方式
    public List<Coder> findByAge(Integer age);
}
```

5.实现一个CoderController，实现增删改查。

```java
@RestController
public class CoderController {
    @Autowired
    private CoderRepository coderRepository;

    //1.Get方式请求，查询所有程序员信息
    @GetMapping(value = "/coders")
    public List<Coder> coderList(){
        return coderRepository.findAll();
    }
    //2.Post方式，增加程序员
    @PostMapping(value = "/coders")
    public Coder CoderAdd(@RequestParam("name")String name,@RequestParam("age")Integer age){
        Coder coder = new Coder();
        coder.setAge(age);
        coder.setName(name);
        return  coderRepository.save(coder);
    }
    //3.通过id查询一个人
    @GetMapping(value = "/coders/{id}")
    public Coder CoderFindOne(@PathVariable("id")Integer id){
        return coderRepository.findOne(id);
    }
    //4.通过id删除一个人
    @DeleteMapping(value = "/coders/{id}")
    public void CoderDelete(@PathVariable("id")Integer id){
        coderRepository.delete(id);
    }
    //5.更新一个人,用Postman模拟的时候注意：用x-www-form-urlencoded
    @PutMapping(value = "/coders/{id}")
    public Coder CoderUpdateOne(@PathVariable("id")Integer id, @RequestParam("name") String name, @RequestParam("age")Integer age){
        Coder coder = new Coder();
        coder.setId(id);
        coder.setName(name);
        coder.setAge(age);
        return coderRepository.save(coder);
    }
    //6.扩展Jpa接口，按照年龄查找
    @GetMapping(value = "/coder/age/{age}")
    public List<Coder> CoderFindAll(@PathVariable("age")Integer age){
        return coderRepository.findByAge(age);
    }

}
```

6.实现mysql的事务

- 首先新建一个Service类:CoderService

```java
@Service
public class CoderService {
    @Autowired
    CoderRepository coderRepository;

    @Transactional
    public void insertTwo(){
        Coder coderA = new Coder();
        coderA.setAge(101);
        coderA.setName("1");
        coderRepository.save(coderA);

        Coder coderB = new Coder();
        coderB.setAge(102);
        coderB.setName("102");
        coderRepository.save(coderB);

    }
}
```

- 在CoderController中自动载入coderService

```java
@Autowired
private CoderService coderService;
```

- 在CoderController调用service。

```java
//7.使用事务，同时插入两个人的数据
@PostMapping(value = "coder/two")
public void coderTwo(){
    coderService.insertTwo();
}
```

7.使用`@Query`实现自定义sql查询

- 在`CoderRepository`实现下面代码

```java
@Query("select p from Coder p where p.id = (select max(p2.id) from Coder p2)")
    Coder getMaxIdCoder();
```

- 在`CoderController`中使用`getMaxIdCoder`方法

```java
//8.自定义sql语句查询
@GetMapping(value = "/coder/find")
public Coder CoderFindByTask(){
    return coderRepository.getMaxIdCoder();
}
```

## Spring Boot MyBatis -- 单数据源

基于注解方式的Mybatis其实和JPA很类似，不过mybatis不提供自动创建表的操作。这点上jpa更好些。

> 我的demo程序，在我的github上：[spring-boot-mybatis-mysql](https://github.com/nezha/spring-boot-mybatis-mysql)
> 
> 引用博客：
> [Spring Boot + MyBatis + MySQL 整合](http://www.jianshu.com/p/c2444ddd2de9)--简书 FlySheep_ly
> 
> 程序员DD：[Spring Boot整合MyBatis](http://blog.didispace.com/springbootmybatis/)
>          
>          [Spring Boot中使用MyBatis注解配置详解](http://blog.didispace.com/mybatisinfo/)

1.引入依赖

```xml
<!-- 添加 MyBatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.2.0</version>
</dependency>
<!-- 添加 MySQL -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.41</version>
</dependency>
```

2.application.properties中配置mysql的连接配置

```xml
spring.datasource.url=jdbc:mysql://localhost:3306/test01
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

3.创建映射对象User

关于序列化的实现，最好还是实现一下。

```java
import java.io.Serializable;

public class User implements Serializable{
    private static final long serialVersionUID = -5554561712056198940L;

    private Long id;
    private String name;
    private Integer age;

    public User(){
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

}
```

4.创建User映射的操作UserMapper

关于Mapper的更多操作，请参考mybatis官网

```java
/**
 * Created by nezha on 2017/4/26.
 */

@Mapper
public interface UserMapper {
    /**
     * 添加操作，返回新增元素的 ID
     * @param User
     */
    @Insert("insert into person(name,age) values(#{name},#{age})")
    @Options(useGeneratedKeys = true, keyColumn = "id", keyProperty = "id")
    void insert(User user);
    /**
     * 查询所有
     * @return
     */
    @Select("select id,name,age from person")
    List<User> selectAll();

    @Select("SELECT * FROM USER WHERE NAME = #{name}")
    User findByName(@Param("name") String name);
}
```

5.调用测试

发现Restful风格真是好习惯，很好用很实用。

```java
@EnableTransactionManagement
@RestController
public class TestController {

    @Autowired
    private UserMapper userMapper;

    @GetMapping(value = "/test/{name}")
    public User findOne(@PathVariable("name")String name){
        return userMapper.findByName(name);
    }
}
```

###可能出现的问题：

> mybatis+spring boot, mapper 提示Could not autowire. 一直提示不能加载

![](http://images2015.cnblogs.com/blog/977217/201607/977217-20160718164109076-508374758.png)

- 解决方案

修改idea配置，将spring 的severity的值设置为"warning", 如下：
![](http://images2015.cnblogs.com/blog/977217/201607/977217-20160719183149701-931010371.jpg)

如果想进一步缩小修改范围的话：

```
Alt + Enter quick fix or change settings
Settings - Editor - Inspections - Spring - Spring Core - Code - Autowiring for Bean Class - warning
```

### 关于多源配置的问题：

1.[Spring Boot 整合 Mybatis 实现 Druid 多数据源详解](http://www.jianshu.com/p/c6770b1466b8)

2.[springboot+mybatis多数据源最简解决方案](http://www.ityouknow.com/springboot/2016/11/25/springboot(%E4%B8%83)-springboot+mybatis%E5%A4%9A%E6%95%B0%E6%8D%AE%E6%BA%90%E6%9C%80%E7%AE%80%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.html)

## Spring Boot RabbitMQ

> 我的demo程序：[spring-boot-RabbitMQ](https://github.com/nezha/spring-boot-RabbitMQ)

> [Spring Boot RabbitMQ 入门](http://www.jianshu.com/p/22687e6e2a38)--这篇文章写得不错,关于RabbitMQ的三种Exchange方式讲的很好。不过代码不是很简洁。

> [RabbitMQ详解](http://www.ityouknow.com/springboot/2016/11/30/springboot(八)-RabbitMQ详解.html)--[纯洁的微笑的文章代码很简洁](https://github.com/ityouknow)

### 安装与配置

> [翟永超-Spring boot系列教程](http://blog.didispace.com/spring-boot-rabbitmq/)

### RabbitMQ的三种Exchange方式

**1.Direct Exchange**

![](http://upload-images.jianshu.io/upload_images/3239635-f350dec07f1883ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果 routing key 匹配, 那么Message就会被传递到相应的queue中。其实在queue创建时，它会自动的以queue的名字作为routing key来绑定那个exchange。

**2.Fanout Exchange**

![](http://upload-images.jianshu.io/upload_images/3239635-6068a7de288f4e53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

只需要简单的将队列绑定到交换机上。一个发送到交换机的消息都会被转发到与该交换机绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。Fanout交换机转发消息是最快的。

**3.Topic Exchange**

![](http://upload-images.jianshu.io/upload_images/3239635-d60da0cd0754fdfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将路由键和某模式进行匹配。此时队列需要绑定要一个模式上。符号“#”匹配一个或多个词，符号“*”匹配不多不少一个词。因此“audit.#”能够匹配到“audit.irs.corporate”，但是“audit.*”

### 实例讲解

> 三种方式最主要的文件就三个：
> 
> 1.sender的配置
> 
> 2.receiver的配置
> 
> 3.rabbitConfig的配置

下面，我们通过在Spring Boot应用中整合RabbitMQ，并实现一个简单的发送、接收消息的例子来对RabbitMQ有一个直观的感受和理解。

在Spring Boot中整合RabbitMQ是一件非常容易的事，因为之前我们已经介绍过Starter POMs，其中的AMQP模块就可以很好的支持RabbitMQ，下面我们就来详细说说整合过程：

1.pom依赖引入

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

2.连接配置

```xml
spring.application.name=rabbitmq-hello
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

3.创建消息生产者Sender

创建消息生产者Sender。通过注入AmqpTemplate接口的实例来实现消息的发送，AmqpTemplate接口定义了一套针对AMQP协议的基础操作。在Spring Boot中会根据配置来注入其具体实现。在该生产者，我们会产生一个字符串，并发送到名为hello的队列中。

```java
@Component
public class Sender {
    @Autowired
    AmqpTemplate amqpTemplate;
    public void send() {
        String context = "hello " + new Date();
        System.out.println("Sender : " + context);
        this.amqpTemplate.convertAndSend("hello", context);
    }
}
```

4.创建消息消费者Receiver

创建消息消费者Receiver。通过@RabbitListener注解定义该类对hello队列的监听，并用@RabbitHandler注解来指定对消息的处理方法。所以，该消费者实现了对hello队列的消费，消费操作为输出消息的字符串内容。

```java
@Component
@RabbitListener(queues = "hello")
public class Receiver {
    @RabbitHandler
    public void process(String hello) {
        System.out.println("Receiver1 : " + hello);
    }
}
```

5.创建RabbitMQ的配置类RabbitConfig

创建RabbitMQ的配置类RabbitConfig，用来配置队列、交换器、路由等高级信息。这里我们以入门为主，先以最小化的配置来定义，以完成一个基本的生产和消费过程。

```java
@Configuration
public class RabbitConfig {


    //1.配置一个名为hello的一对一的消息队列
    @Bean
    public Queue helloQueue() {
        return new Queue("hello");
    }
}
```

5.单元测试:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = HelloApplication.class)
public class HelloApplicationTests {
    @Autowired
    private Sender sender;
    @Test
    public void hello() throws Exception {
        sender.send();
    }
}
```

##  Spring Boot mongodb

> spring boot mongodb 的demo程序:[spring-boot-mongodb](https://github.com/nezha/spring-boot-mongodb)

### 安装与配置

- mongodb的安装与配置: [CentOS 6.5下通过yum安装MongoDB记录](http://blog.csdn.net/testcs_dn/article/details/50698572)
- 用户管理参考：[mongodb 3.2 用户权限管理配置](http://www.cnblogs.com/mymelody/p/5906199.html)
- Mac下的mongodb安装与配置: [Mac 上安装MongoDB](http://www.jianshu.com/p/dd0c39bf7be4)

1、进入mongodb的shell ： mongo

2、切换数据库： use admin

3、添加用户，指定用户的角色和数据库：

```
db.createUser(  
  { user: "admin",  
    customData：{description:"superuser"},
    pwd: "admin",  
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]  
  }  
)  
user字段，为新用户的名字；
pwd字段，用户的密码；
cusomData字段，为任意内容，例如可以为用户全名介绍；
roles字段，指定用户的角色，可以用一个空数组给新用户设定空角色。在roles字段,可以指定内置角色和用户定义的角色。
```

4、查看创建的用户 ： show users 或 db.system.users.find()

5、启用用户权限：

修改配置文件，增加配置：

```
$ vim /etc/mongod.conf
security:
  authorization: enabled
```

6.重新启动mongodb

```
sudo service mongod restart
```

7.使用用户管理账户登录认证

```
use admin
db.auth('admin', 'admin')
```

8.远程登陆

```shell
mongo 123.xxx.xxx.xxx:27017/amdin -uadmin -padmin
```

> 第一个admin:指代数据库
> 第二个admin:指代用户名
> 第三个admin:指代密码

### spring boot 集成 mongodb

1.引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

2.创建存储的实体

创建要存储的User实体，包含属性：id、username、age

```java
public class User {

    @Id
    private String id;

    private String username;
    private Integer age;

    public User(String username, Integer age) {
//        this.id = id;
        this.username = username;
        this.age = age;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "username:"+username+"--age:"+age;
    }
}
```

3.实现User的数据访问对象

实现User的数据访问对象：UserRepository

```java
public interface UserRepository extends MongoRepository<User, String> {

    User findByUsername(String username);

}
```

4.配置mongodb的连接

```xml
#mongodb3.X的配置
spring.data.mongodb.uri=mongodb://admin:admin@123.206.xxx.xxx:27017/test
#mongodb2.x的配置
spring.data.mongodb.host=localhost spring.data.mongodb.port=27017
```

5.在单元测试中调用

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test05ApplicationTests{
    @Autowired
    private UserRepository userRepository;

    @Test
    public void test() {
        userRepository.deleteAll();
        // 创建三个User，并验证User总数
        userRepository.save(new User("didi", 30));
        userRepository.save(new User("mama", 40));
        userRepository.save(new User("kaka", 50));
        Assert.assertEquals(3, userRepository.findAll().size());

        // 删除一个User，再验证User总数
        User u = userRepository.findByUsername("didi");
        System.out.println(u.toString());
        userRepository.delete(u);
        Assert.assertEquals(2, userRepository.findAll().size());
    }

}
```

### 出现的问题

1.发现运行成功，但是查不到数据

> 主要是自己理解错误，mongodb数据库下还有collection(相当于表).然后针对每一个Database下可能有多个collection

##  Spring Boot redis

> redis的demo程序：[nezha/spring-boot-redis](https://github.com/nezha/spring-boot-redis)

- redis的配置：[CentOS6.5下Redis安装与配置](http://blog.csdn.net/ludonqin/article/details/47211109)
- redis开启远程连接[redis开启远程访问](http://www.cnblogs.com/liusxg/p/5712493.html)
- redis远程连接`redis-cli -h 123.206.xxx.xxx -p 6379`

1.引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-redis</artifactId>
</dependency>
```

2.参数配置

```xml
# REDIS (RedisProperties)
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=123.206.xxx.xxx
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=-1
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=16
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=0
```

3.测试访问

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test04ApplicationTests {

    @Autowired
    StringRedisTemplate stringRedisTemplate;

    @Test
    public void test() throws Exception {
        // 保存字符串
        stringRedisTemplate.opsForValue().set("aaa", "111");
        Assert.assertEquals("111", stringRedisTemplate.opsForValue().get("aaa"));
    }

}
```

## Spring Boot定时任务

> 定时任务demo程序:[spring-boot-schedule](https://github.com/nezha/spring-boot-schedule)

1.pom包配置

> 这部分基本的spring boot启动项就行，没有什么特别的依赖包

2.启动类启用定时

```java
@EnableScheduling
@SpringBootApplication
public class ScheduleApplication {

    public static void main(String[] args) {
        SpringApplication.run(ScheduleApplication.class, args);
    }
}
```

3.创建定时任务实现类

定时任务一

```java
/**
 * Created by nezha on 2017/4/28.
 */
@Component
public class SchedulerTask {
    private int count = 0;

    @Scheduled(cron="*/6 * * * * ?")
    private void process(){
        System.out.println("this is scheduler task runing  "+(count++));
    }
}
```

定时任务二

```java
/**
 * Created by nezha on 2017/4/28.
 */
@Component
public class SchedulerTask2 {
    private static SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    @Scheduled(fixedRate = 6000)
    public void reportCurrentTime() {
        System.out.println("现在时间：" + dateFormat.format(new Date()));
    }
}
```

4.参数说明

`@Scheduled` 参数可以接受两种定时的设置，一种是我们常用的`cron="*/6 * * * * ?"`,一种是 `fixedRate = 6000`，两种都表示每隔六秒打印一下内容。

**fixedRate 说明**

- `@Scheduled(fixedRate = 6000)` ：上一次开始执行时间点之后6秒再执行
- `@Scheduled(fixedDelay = 6000)` ：上一次执行完毕时间点之后6秒再执行
- `@Scheduled(initialDelay=1000, fixedRate=6000)` ：第一次延迟1秒后执行，之后按fixedRate的规则每6秒执行一次

## Spring Boot相关技术

### 自定义图标

[Spring Boot自定义Banner](http://www.jianshu.com/p/e719f57191d9)---自定义图标

### 部署spring项目

1.IntelliJ Idea中直接运行

2.mvn spring-boot:run

3.mvn install >>> 会生成jar文件，执行jar文件。

### spring boot测试的时候，怎么单元测试

1.使用`@SpringBootTest`
文件主要实在test目录下

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test05ApplicationTests{
    @Autowired
    private UserRepository userRepository;

    @Test
    public void test() {
        userRepository.deleteAll();
        // 创建三个User，并验证User总数
        userRepository.save(new User("didi", 30));
        userRepository.save(new User("mama", 40));
        userRepository.save(new User("kaka", 50));
    }
}
```

2.使用`@SpringBootApplication`

在主目录下com.nezha/

```java
@SpringBootApplication
public class Application implements CommandLineRunner {
    @Autowired
    private CustomerRepository repository;
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    @Override
    public void run(String... args) throws Exception {
        repository.deleteAll();
        // save a couple of customers
        repository.save(new Customer("Alice", "Smith"));
        repository.save(new Customer("Bob", "Smith"));
    }
}
```

3.使用`RestController`

这种方式很方便，很好用，有些时候我会用。


## 待研究学习框架

1. Spring Boot Dubbo
2. Spring Boot Druid