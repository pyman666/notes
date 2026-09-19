
- [入门](#入门)
- [application](#application)
- [profile](#profile)
- [加载顺序](#加载顺序)
- [切换内置web服务器](#切换内置web服务器)
- [Condition](#condition)
- [Import](#import)
- [EnableAutoConfigration](#enableautoconfigration)

## 入门

## application

- SpringBoot配置文件类型：properties / yml / yaml

- 默认配置文件名称：application

- 配置优先级：properties > yml > yaml
```yaml
name: eric
age: 18

person:
  name: hening
  age: ${age}

address:
  - BeiJing
  - ShangHai
```
```java
@RestController
public class HelloController {

    @Value("${name}")
    private String name0;

    @Value("${person.name}")
    private String name1;

    @Value("${person.age}")
    private int age1;

    @Value($"address[0]")
    private String address0;

    @Autowired
    private Environment env;

    private String address1 = env.getProperty("address[0]");

    @Autowired
    private  Person person;

    @RequestMapping("/hello")
    public String hello() {
        return "hello world";
    }

}
```
```java
@Component
@ConfigurationProperties(prefix="person")
public class Person {

    private String name;
    private int age;

}
```
## profile

用来完成不同环境下配置动态切换

- 方式一：
```yaml
spring:
  profiles:
  	active: dev
```
```yaml
server:
	port: 8080
```
- 方式二（已过时）：
```yaml
spring:
	profiles:
		active: dev

---
server:
	port: 8080

spring:
	profiles: dev

---
server:
	port: 8080

spring:
	profiles: test
```
## 加载顺序

SpringBoot启动时，会从以下位置加载配置文件（优先级亦如下）：

- 当前项目下的`/config`目录下

- 当前项目根目录

- classpath的`/config`目录

- classpath的根目录

# 自动配置

## 切换内置web服务器

SpringBoot的web环境默认使用tomcat作为内置服务器，总共提供了4个。

切换：pom文件中先排除tomcat依赖，再引入（如jetty）依赖

## Condition

实现选择性地（满足一定条件）创建Bean

自定义条件：

- 定义条件类：自定义类实现`Condition`接口，重写`matches`方法，matches方法两个参数：

- `context`：上下文对象，可以获取属性值，获取类加载器，获取BeanFactory等；

- `metadata`：元数据对象，用于获取注解属性

- 判断条件：在初始化Bean时，使用`@Conditional`（条件类.class）注解
```java
@Configuration
public class MyConfig {
    @Bean
    //@Conditional(MyCondition.class)
    @MyConditionOnClass("redis.clients.jedis.Jedis")
    public Me me() {
        return new Me();
    }
}
```
```java
public class Application() {
    public static void main(String[] args) {
        // 启动SpringBoot应用，返回Spring的IOC容器
        ConfiguratbaleApplicationContext context = SpringApplication.run(SpringbootEnableApplication.class, args);

        MyConfig config = context.getBean("me");
    }
}
```
```java
public class MyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        try {
            Class<?> cls = Class.forName("redis.clients.jedis.Jedis");
            return true;
        } catch (ClassNotFoundException e) {
            return false;
        }
    }
}
```
```java
public class NewCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Map<String, Object> map = metadata.getAnnotationAttributes(MyConditionOnClass.class.getName());
        String[] value = (String[]) map.get("value");
        try {
            for (String className: value) {
                Class<?> cls = Class.forName(className);
            	return true;
            }
        } catch (ClassNotFoundException e) {
            return false;
        }
    }
}
```
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(NewCondition.class)
public @interface MyConditionOnClass {
    String[] value;
}
```
SpringBoot提供的常用条件注解：

- `@Conditional`

- `@ConditionalOnProperty`：配置文件中是否有对应属性和值

- `@ConditionalOnClass`：环境中是否有对应字节码文件

- `@ConditionalOnMissingBean`：环境中没有对应Bean

## Import

- `@ComponentScan`：要扫描的包（默认扫描包范围：当前引导类所在包及其子包）

- `@Import`： 要引入的类，这些（一般外部）类会被Spring创建并放入IOC容器

`@Import`4种用法：

- 导入Bean：直接导入某个类

- 导入配置类：配置类（可以不加`@Configration`）中加了`@Bean`的都会被导入

- 导入`ImportSelector`实现类，一般用于加载配置文件中的类

- 导入`ImportBeanDefinitionRegistrar`实现类
```java
//@ComponentScan("com.demo.config")
//@Import(MyConfig.class)
//@Import(MySelector.class)
//@Import(MyRegistrar.class)
@EnableMyConfig
public class Application() {
    public static void main(String[] args) {
        ConfiguratbaleApplicationContext context = SpringApplication.run(SpringbootEnableApplication.class, args);
        MyConfig config = context.getBean(MyConfig.class);
    }
}
```
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(MyConfig.class)
public @interface EnableMyConfig {}
```
```java
public class MySelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importintClassMetadata) {
        return new String[] {"com.demo.config.MyConfig"};
    }
}
```
```java
public class MyRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importintClassMetadata, BeanDefinitionRegistry registry) {
        AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(MyConfig.class).getBeanDefinition();
        registry.registerBeanDefinition("myConfig", beanDefinition);
    }
}
```
## EnableAutoConfigration

- `@EnableAutoConfigration`内部使用`@Import`（`AutoConfigurationImportSelector.class`）来加载配置类；

- 配置文件位置：/META-INF/spring.factories，该配置文件中定义了大量的配置类，当SpringBoot应用启动时，会自动加载这些配置类，初始化Bean；

- 并不是所有的Bean都会被初始化，在配置类中使用Condition来加载满足条件的Bean

SpringBoot提供了很多Enable开头的注解，都是用于动态启用某些功能。其底层眼里都是使用`@Import`注解导入一些配置类，实现Bean的动态加载，常用：

- @EnableConfigurationProperties：从配置文件类来加载
```java
@Configuration
@EnableConfigurationProperties(RedisProperties.class)
public class RedisAutoConfiguration {
    @Bean
    public Jedis jedis(RedisProperties redisProperties) {
        return new Jedis(redisProperties.getHost(), redisProperties.getPort());
    }
}
```
```java
@ConfigurationProperties(prefix="redis")
public class RedisProperties {
    private String host = "localhost";
    private int port = 6379;
    // get、set ...
}
```
# 监听机制

SpringBoot在项目启动时，会对几个监听器进行回调，可以实现这些监听器接口，在项目启动时完成一些操作。

- ApplicationContextInitializer

- SpringApplicationRunListener

- CommandLineRunner

- ApplicationRunner
```java
public class MyApplicationContextInitializer implements ApplicationContextInitializer {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {}
}
```
```java
public class MySpringApplicationRunListener implements SpringApplicationRunListener {

	public MySpringApplicationRunListener(SpringApplication application, String[] args) {}

    Override
    public ...() {}  // 多为跟生命周期相关的方法
}
```
```java
public class MyCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {}
}
```
```java
public class MyApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {}
}
```
# 监控

SpringBoot自带监控功能Actuator，可以帮助实现对程序内部运行情况的监控，如监控状态、Bean加载情况、配置属性、日志信息等。

启用：在POM中引入依赖，访问 http://host:port/acruator
