---
title: SpringBoot 快速入门
date: 2020-05-15 09:48:45
tags: [Java, SpringBoot]
---

SpringBoot 快速入门学习笔记。

<!--more-->

# 快速入门

## 创建项目
打开 IDEA 新建项目，选择 `Spring Initializr`，勾选 `Web` 依赖。

## SpringBoot 配置

### 两种配置文件

SpringBoot 的配置文件有两种：`application.properties` 和 `application.yml`。两种文件效果一样，只是写法不同。

`application.properties` 文件配置：

```yaml
server.port=8080
server.servlet.context-path=/test
```

`application.yml` 文件配置：

```yaml
server:
  port: 8081
  servlet:
    context-path: /test
```

> 注意：冒号后面必须加个空格，不能写成 port:8080，需要在 port: 和 8080 之间加空格。

对比来看 `application.yml` 文件写法更精简，建议使用。

### 属性配置

可在 `application.yml` 配置文件里自定义配置信息并在项目中读取。

#### 单个属性读取

配置信息 `name` 和 `age`

```yaml
name: A
age: 20
```

在 `Controller` 中通过 `@Value("${age}")` 注解读取配置文件中的属性。

```java
@RestController
public class HelloController {

    @Value("${name}")
    private String name;
    
    @Value("${age}")
    private Integer age;
    
    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String say() {
        return name + age;
    }
}
```

#### 通过对象多属性一起读取

`application.yml`

```yaml
girl:
  name: B
  age: 20
```

新建类 `Girl.java`

```java
@Component
@ConfigurationProperties(prefix = "girl")
@Data
public class Girl {
    private String name;
    private Integer age;
}
```

Controller 中使用

```java
@RestController
public class HelloController {

    @Autowired
    private Girl girl;

    @RequestMapping(value = "/sayProperties", method = RequestMethod.GET)
    public String sayProperties() {
        return girl.getName() + girl.getAge();
    }
}
```

### 多环境配置

配置开发环境和生产环境

新建 `application-dev.yml` 作为开发环境配置

```yaml
server:
  port: 8082

girl:
  name: C
  age: 24
```

新建 `application-prod.yml` 作为生产环境配置

```yaml
server:
  port: 8081

girl:
  name: B
  age: 20
```

修改 `application.yml`，配置为开发环境

```yaml
spring:
  profiles:
    active: dev
```

如需配置为生产环境，将 `active: dev` 改为 `active: prod`

```yaml
spring:
  profiles:
    active: prod
```

## 日志

- Slf4j
- LogBack

Log 的配置

## Controller 的使用

   - **@Controller** 处理 http 请求
   - **@RestController** Spring4 之后新加的注解，原来返回 json 需要 @ResponseBody 配合  @Controller
   - **@RequestMapping** 配置 url 映射
   - **@PathVariable** 获取 url 中的数据
   - **@RequestParam** 获取请求参数的值
   - **@GetMapping** 组合注解

### @RestController

```java
@RestController
public class HelloController {
}
```

`@RestController` = `@Controller` + `@ResponseBody` 组成。等号右边两位简单介绍两句，就明白我们 `@RestController` 的意义了：

- `@Controller` 将当前修饰的类注入 SpringBoot IOC 容器，使得从该类所在的项目跑起来的过程中，这个类就被实例化。当然也有语义化的作用，即代表该类是充当 Controller 的作用。

- `@ResponseBody` 它的作用简短截说就是指该类中所有的 API 接口返回的数据，不管你对应的方法返回 Map 或是其他 Object，它会以 Json 字符串的形式返回给客户端。

  > 尝试了一下，如果返回的是 String 类型，则仍然是 String。

```java
@RestController
public class HelloController {

    @RequestMapping(value = "test", method = RequestMethod.GET)
    public Map testGet() {
        return new HashMap<String, String>() {{
            put("name", "SpringBoot");
        }};
    }

    @RequestMapping(value = "testStr", method = RequestMethod.GET)
    public String testGetStr() {
        return "OK";
    }

}
```

这部分代码对于 `Map` 返回则是 `JSON String`，对于 `String` 则仍然是 `String`。

当将 `@RestController` 换成 `@Controller` 之后，对于返回类型为 `Map` 的结果如下：

```text
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Fri May 15 09:33:57 CST 2020
There was an unexpected error (type=Internal Server Error, status=500).
Circular view path [hello]: would dispatch back to the current handler URL [/hello] again. Check your ViewResolver setup! (Hint: This may be the result of an unspecified view, due to default view name generation.)
```

从报错可以看见，当 `@Controller` 修饰的时候，Spring 以为会返回一个 View（也就是 MVC 中的 C），但是返回的东西却是一个 Map。

### @RequestMapping

#### 给类配置访问路径

`@RequestMapping("/hello")`

url 前缀，一般跟类名相似。

#### 给方法配置多个访问路径

给 value 配置多个路径的集合

```java
@RequestMapping(value = {"/happy", "/hi"}, method = RequestMethod.GET)
public String sayHi() {
    return "happy or Hi";
}
```

访问 `http://localhost:8080/hello/happy` 或 `http://localhost:8080/hello/hi` 均可进入方法 `sayHi()`。

### @PathVariable

用来获取 url 中的参数

```java
@RequestMapping(value = "/go/{id}", method = RequestMethod.GET)
public String go(@PathVariable("id") String id) {
    return "id: " + id;
}
```
访问地址：`http://localhost:8080/hello/go/12`

`id` 也可以放前面，效果一样。

### @RequstParam

获取请求参数的值

```java
@RequestMapping(value = "/come", method = RequestMethod.GET)
public String getRequestParam(@RequestParam("id") String id) {
    return "id: " + id;
}
```

访问地址：`http://localhost:8080/hello/come?id=123`

给参数加默认值

```java
@RequestMapping(value = "/come2", method = RequestMethod.GET)
public String getRequestParamDefValue(@RequestParam(value = "id", required = false, defaultValue = "0") String id) {
    return "id: " + id;
}
```

当 ` id` 不传时默认是 0。

### @GetMapping

简化 `@RequestMapping` 的写法，还有 `@PostMapping` 等。

## 数据库操作 JPA
JPA（Java Persistence API）定义了一系列的对象持久化的标准，目前实现这一规范的产品有 `Hibernate`、`TopLink` 等。

### 配置引入 MySQL 和 JPA
修改 `pom.xml` 文件添加 `JPA` 和 `MySQL` 依赖。

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

修改 `application.yml` 文件，配置 `JPA` 和 `MySQL`

```xml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: 123456
    url: jdbc:mysql://localhost/db_name?characterEncoding=utf-8&useSSL=false
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

> 注：url 中最后的 db_name 是数据库名字

ddl-auto 可选参数有五种：
- create 启动时删数据库中的表，然后创建，退出时不删除数据表。
- create-drop 启动时删数据库中的表，然后创建，退出时删除数据表。如果表不存在报错。
- update 如果启动时表格式不一致则更新表，原有数据保留。
- none 不进行配置
- validate 项目启动表结构进行校验。如果不一致则报错。

### 创建数据库和表

创建数据库，建数据库时编码应选用 `utf-8 utf8mb4`，以便能存储表情符号等。

然后在项目中新建类 User：

```java
@Entity
@Data
@NoArgsConstructor  // 需要有空参构造函数
public class User {

    @Id
    @GeneratedValue
    private Integer id;

    private String name;
    private Integer age;
}
```

运行项目，数据库会自动创建表 user。

### 增删改查

创建 UserRepository

```java
public interface UserRepository extends JpaRepository<User, Integer> {

    List<User> findAllByAge(Integer age);
}
```

创建 Controller 类 UserController

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserRepository repository;

    /**
     * 查询所有用户
     *
     * @return
     */
    @GetMapping(value = "/users")
    public List<User> getAll() {
        return repository.findAll();
    }

    /**
     * 添加一个用户
     *
     * @param name
     * @param age
     * @return 返回新添加的对象
     */
    @PostMapping(value = "/add")
    public User add(@RequestParam("name") String name,
                    @RequestParam("age") Integer age) {

        User User = new User();
        User.setAge(age);
        User.setName(name);

        return repository.save(User);
    }

    /**
     * 更新
     *
     * @param id
     * @param name
     * @param age
     * @return
     */
    @PutMapping(value = "/{id}")
    public User update(@PathVariable("id") Integer id,
                       @RequestParam("name") String name,
                       @RequestParam("age") Integer age) {
        User User = new User();
        User.setId(id);
        User.setName(name);
        User.setAge(age);

        return repository.save(User);
    }

    /**
     * 通过 id 查询一个用户
     *
     * @param id
     * @return
     */
    @GetMapping(value = "/{id}")
    public User findOne(@PathVariable("id") Integer id) {
        return repository.findById(id).get();
    }

    /**
     * 删除
     *
     * @param id
     */
    @DeleteMapping(value = "/{id}")
    public void deleteById(@PathVariable("id") Integer id) {
        repository.deleteById(id);
    }

    /**
     * 通过年龄查
     *
     * @param age
     * @return
     */
    @GetMapping(value = "/age/{age}")
    public List<User> findByAge(@PathVariable("age") Integer age) {
        return repository.findAllByAge(age);
    }
}
```

以上通过 JPA 完成了增删改查。一般地，我们把业务逻辑放在 Service 中。


# 开发

- DAO 层设计与开发
- Service 层设计与开发
- Controller 层设计与开发

### 步骤

#### 1. 设计数据库

#### 2. 数据库对象：数据表映射对象

##### 2.1 Entity

```java
@Entity
@Table(name = "tb_(name)_")  // 数据库表和数据对象不同时加此注解
@DynamicUpdate  // 更新日期
@Data
public class CccEntity {

    @Id  // 主键
    @GeneratedValue  // 自增
    private Integer categoryId;
}
```

@Transient：忽略

##### 2.2 DTO——数据传输对象

各个层里传输用

#### 3. DAO

```java
public interface XxxRepository extends JpaRepository<entity, 主键> {
}
```

#### 4. Service

##### 4.1

```java
public interface SssService {
}
```

##### 4.2 实现

```java
@Service
public class SssServiceImpl implements SssService {

    @Autowired
    private XxxRepository repository;

    @Override
    public CccEntity findOne(Integer categoryId) {
        return repository.findOne(categoryId);
    }

    @Override
    public List<CccEntity> findAll() {
        return repository.findAll();
    }
}
```


对象的属性拷贝：`BeanUtils.copyProperties(source, target);`


列表分页：Page，Pageable（接口），PageRequest

可将分页列表数据进行封装和转换

```java
@Data
public class CountPageData<T> {

    private Long totalCount;

    private List<T> items;
    private boolean hasMore;

    public CountPageData(List<T> items, boolean hasMore, Number totalCount) {
        this.items = items;
        this.hasMore = hasMore;
        this.totalCount = totalCount.longValue();
    }

    public static <T> CountPageData<T> of(Page<T> page) {
        return new CountPageData<>(page.getContent(), page.hasNext(), page.getTotalElements());
    }

}

// use
CountPageData cpd = CountPageData.of(Page 对象);
```

#### 5. Controller

##### 5.1 Controller
```java
@RestController
@RequestMapping("/buyer/order")
@Slf4j
public class BuyerOrderController {

    @Autowired
    private OrderService orderService;

    @GetMapping("/detail")
    @PostMapping("/cancel")
}
```

##### 5.2 Form

**@JsonProperty("name")**：返回给使用者（前端）时的对象名

```java
@JsonProperty("created_at")
@JsonSerialize(using = DateTimeSerializer.class)
@JsonDeserialize(using = DateTimeDeserializer.class)
private LocalDateTime createdAt;
```

#### 6. 测试

以上三步都需要单元测试

```java
@SpringBootTest

// 断言
Assert.asseretNotMull(result);
Assert.assertNotEquals(0, actual);
Assert.assertTrue("message", condition);
```

