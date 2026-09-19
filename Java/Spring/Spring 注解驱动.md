
- [概览](#概览)
- [组件注册](#组件注册)
  - [配置类](#配置类)
  - [包扫描](#包扫描)
  - [作用域](#作用域)
  - [懒加载](#懒加载)
  - [按条件注册](#按条件注册)
  - [快速导入](#快速导入)
  - [工厂Bean](#工厂bean)
- [生命周期](#生命周期)
  - [初始化🆚销毁](#初始化销毁)
- [属性赋值](#属性赋值)
- [自动装配](#自动装配)
  - [自动注入](#自动注入)
  - [底层注入](#底层注入)
  - [动态注入](#动态注入)

## 概览

## 组件注册

给IOC容器中注册组件：

- 自己标注的：`@Component`、`@Controller`、`@Service`、`@Repository`等

- 配置类：`@Configuration`、`@Bean`

- 包扫描：`@ComponentScan`指定的范围

- 快速导入：`@Import`

- 工厂Bean：`FactoryBean`

### 配置类

配置文件xml -> 配置类`@Configuration`

标签`<bean>` -> 注解`@Bean`
```xml
<beans xmlns=...>
  <bean id="person", class="com.demo.bean.Person">
    <property name="name", value="eric"/>
  	<property name="age", value="18"/>
  </bean>
</beans>
```
```java
@Configuration  // 告诉Spring这是一个配置类
public class MainConfig {
    @Bean  // 给容器中注册一个Bean，class为返回值类型，id默认为方法名
    // @Bean("eric")  指定id
    public Person person() {
        return new Person("eric", 18);
    }
}
```
```java
public class MainTest {
    public static void main(Strings[] args) {

        // ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
        ApplicationContext ctx = new AnnotationConfigApplicationContext(MainConfig.class);

        // Person bean = (Person) ctx.getBean("person");
        Person bean = (Person) ctx.getBean(Person.class);
        String[] namesForType = ctx.getBeanNamesForType(Person.class);
        Map<String, Person> map = ctx.getBeansOfType(Person.class);
    }
}
```
### 包扫描

标签`<context:component-sacn>` -> 注解`@ComponentScan`

过滤规则：

- `FilterType.ANNOTATION` 按注解过滤；

- `FilterType.ASSIGNABLE_TYPE` 按照给定的类（包括子类、实现类等）；

- `FilterType.CUSTOM` 按自定义规则（`TypeFilter`的实现类）

- `FilterType.REGEX` 按正则表达式

- `FilterType.ASPECTJ` 按ASPECTJ表达式
```xml
<beans>
  <context:component-sacn base-package="com.demo" use-default-filters="false"/>
</beans>
```
```java
@Configuration
@ComponentScan(value="com.demo", excludeFilters={
    @Filter(type=FilterType.ANNOTATION, classes={Controller.class})  // 排除加了Controller注解的
})
public class MainConfig { }
```
```java
@Configuration
@ComponentScans(value={
    @ComponentScan(value="com.demo", includeFilters={...}, useDefaultFilters=false),
    @ComponentScan(...),
    ...
})
public class MainConfig { }
```
```java
public class MyFilter implements TypeFilter {
    @Override
    public boolean match(MetadataReader reader, MetadataReaderFactory factory) throws IOException {
        // reader: 读取到的当前正在扫描的类的信息
        // factory: 可以获取其他任何类的信息

        // 获取当前类注解的信息
        AnnotitionMetadata annotationMetadata = reader.getAnnotitionMetadata();
        // 获取当前正在扫描的类的类信息
        ClassMetadata classMetadata = reader.getClassMetadata();
        // 获取当前类资源（类的路径等）
        Resource resource = reader.getResource();

        String className = classMetadata.getClassName();
        if (className.contains("er")) return ture;  // 只匹配类名中包含er的
        return false;
    }
}
```
### 作用域

属性`scope` -> 注解`@Scope`

- `prototype`：多实例，IOC容器启动不会创建对象到容器中，每次获取的时候才会创建

- `singleton`：单例（默认），IOC容器启动就会创建对象到容器，以后每次直接从容器拿（`map.get`）

- `request`：同一次请求创建一个实例

- `session`：同一个session创建一个实例
```xml
<bean id="person" class="com.demo.bean.Person" scope="prototype"/>
```
```java
@Configuration  // 告诉Spring这是一个配置类
public class MainConfig {
    @Bean("eric")
    @Scope("prototype")
    // @Lazy 单例懒加载
    // @Conditional({MyCondition.class}) 根据条件注册
    public Person person() {
        return new Person("eric", 18);
    }
}
```
### 懒加载

`@Lazy` 使单例对象在容器启动的时候先不创建对象，第一次使用的时候创建

### 按条件注册

`@Conditional` 按一定条件判断，满足条件才给容器中注册Bean。可以标在方法上，也可类上
```java
public class MyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext ctx, AnnotatedTypeMetadata meta) {
        // ctx: 判断条件能使用的上下文环境
        // meta：注释信息
        ConfigurableListableBeanFactory factory = ctx.getBeanFactory();  // 获取IOC的bean工厂
        ClassLoader loader = ctx.getClassLoader();  // 获取类加载器
        Environment env = ctx.getEnvironment();  // 获取当前环境信息
        BeanDefinitionRegistry registry = ctx.getRegistry();  // 获取Bean定义的注册类

        String os = env.getProperty("os.name");  // 获取操作系统
        if (os.contains("Windows")) return true;  // 当前是Win才注册
        return false;
    }
}
```
### 快速导入

`@Import`要导入的组件，容器中就会自动注册，id默认是全类名（com.demo.bean.Color）

`ImportSelector`：返回需要导入的组件的全类名数组

`ImportBeanDefinitionRegistrar`：直接自己手工注册
```java
@Configuration
@Import({Color.class, MySelector.class, MyRegistrar.class})
public class MainConfig { }
```
```java
public class MySelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata meta) {
        // 返回值：要导入到容器中的组件全类名
        // meta：当前标注@Import注解的类的所有注解信息

        return new String[]{"com.demo.bean.Blue", "com.demo.bean.Red"};
    }
}
```
```java
public class MyRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata meta, BeanDefinitionRegistry registry) {
        // meta：当前标注@Import注解的类的所有注解信息
        // registry：BeanDefinition注册类

        if (registry.containsBeanDefinition("com.demo.bean.Red")) {
            RootBeanDefinition beanDefinition = new RootBeanDefinition(Blue.class);
            registry.registerBeanDefinition("blue", beanDefinition);
        }
    }
}
```
### 工厂Bean

默认获取的是工厂Bean调用`getObject`方法创建的对象，要取工厂Bean本身需加&
```java
@Configuration
public class MainConfig {
    @Bean
    public ColorFactoryBean colorFactoryBean() {
        return new ColorFactoryBean();
    }
}
```
```java
// 创建一个Spring定义的工厂Bean
public class ColorFactoryBean implements FactoryBean<Color> {

    // 返回Color对象，对象会添加到容器中
    @Override
    public Color getObject() throws Exception {
        return new Color();
    }

    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }

    @Override
    public boolean isSingleton() {
        return false;  // 非单例
    }
}
```
```java
public class MainTest {
    public static void main(Strings[] args) {
        AppicationContext ctx = new AnnotationConfigApplicationContext(MainConfig.class);

        Object bean1 = ctx.getBean("colorFactoryBean");
        // bean1.getClass()  com.demo.bean.Color

        Object bean2 = ctx.getBean("&colorFactoryBean");
        // bean2.getClass()  com.demo.bean.ColorFactoryBean
    }
}
```
## 生命周期

Bean的生命周期：创建 -> 初始化 -> 销毁的过程。

**创建**：

- 单实例：容器启动的时候创建对象；

- 多实例：每次获取的时候创建对象；

**初始化**：

对象创建完成，并赋值好，调用初始化方法；

**销毁**：

- 单实例：容器关闭时调用销毁方法；

- 多实例：容器不会管理这个bean，容器不会调用销毁方法

### 初始化🆚销毁

容器管理bean生命周期，但可以自定义初始化和销毁方法，容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法。

- 直接指定初始化和销毁方法；

- `InitializingBean`接口定义初始化逻辑，`DisposableBean`接口定义销毁逻辑；

- `@PostConstruct`：bean创建并且赋值完成之后执行，`@PreDestroy`在容器销毁bean之前执行；

- `BeanPostProcessor`：bean的后置处理器，在bean初始化前后进行一下处理工作；

法一：
```xml
<bean id="person" class="com.demo.bean.Person" init-method="init" destroy-method="destroy"/>
```
```java
@Configuration
public class MainConfig {
    @Bean(initMethod="init", destroyMethod="destroy")  // 指定初始化和销毁方法
    public Car car() {
        return new Car();
    }
}
```
```java
public class Car {
    public Car() {}
    public void init() {}  // 初始化方法
    public void destroy() {}  // 销毁方法
}
```
法二：
```java
@Component
public class Cat implements InitializingBean, DisposableBean {
    public Cat() {}

    @Override  // 初始化方法
    public void arterPropertiesSet() throws Exception {}

    @Override  // 销毁方法
    public void destroy() throws Exception {}
}
```
法三：
```java
@Component
public class Dog {
    public Dog() {}

    @PostConstruct  // 对象创建并赋值之后调用
    public void init() {}

    @PreDestroy  // 容器移除之前调用
    public void destroy() {}
}
```
法四：
```java
@Component  // 后置处理器加入容器
public class MyProcessor implements BeanPostProcessor {
    @Override  // 初始化前工作
    public Object postProcessBeforeInitialization(Objcet bean, String beanName) throws BeansException {
        // bean 刚创建还没初始化的实例
        // beanName bean的名字
        return bean;
    }

    @Override  // 初始化后工作
    public void postProcessAfterInitialization(Objcet bean, String beanName) throws BeansException {
        return bean;
    }
}
```
## 属性赋值
```xml
<beans>
  <context:property-placeholder location="classpath:person.properties" />
  <bean id="person", class="com.demo.bean.Person">
    <property name="name", value="eric"/>
    <property name="age", value="18"/>
  </bean>
</beans>
```
```java
@Configuration
@PropertySource(value={"classpath:/person.properties"})  // 读取配置文件中的k-v保存到运行的环境变量中
// @PropertySources(value={@PropertySource(...), ...})  多个
public class MainConfig {
    @Bean
    public Person person() {
        return new Person();
    }
}
```
```java
@Component
public class Person {
    //@Value("eric")  直接赋值
    @Value("${person.name}")  // 从配置文件读取
    private String name;

    @Value("#{20 - 2}")  // SpEL表达式
    private int age;
}
```
```
person.name=eric
```
## 自动装配

Spring利用依赖注入（**DI**），完成对IOC容器中各个组件的依赖关系赋值。

### 自动注入

`@Autowired`：（Spring注解）

- 构造器、参数、方法、属性 都能标；

- 默认优先按照**类型**去容器中找对应的组件，找到就赋值；

- 如果找到多个相同类型的组件，再将**属性名**作为组件的id去容器中查找；

- 使用`@Qualifier`指定需要装配的组件的id，而不是使用属性名；

- 默认一定要将属性赋值好，找不到就报错，可以指定`required`属性使之找不到也不报错；

- `@Primary`自动装配的首选的bean，也可以用`@Qualifier`继续指定非首选的bean；

`@Resource`：（Java规范注解）默认按照**组件名称**进行自动装配，不支持`@Primary`、`required`；

`@Inject`：（Java规范注解）和`@Autowired`一样，支持`@Primary`，不支持`required`；

**属性注入**
```java
@Service
public class Boss {
    @Autowired(required=false)  // 非必需
    @Qualifier("car1")  // 直接指定
    // @Resource(name="car1")
    // @Inject
    private Car car;
}
```
```java
@Bean("car2")
@Primary
public class Car { }
```
**方法注入**
```java
@Component
public class Boss {
    private Car car;

    @Autowired  // 方法使用的参数，自定义类型的值从IOC容器中获取
    public void setCar(Car car) {
        this.car = car;  // IOC容器中的car
    }
}
```
**构造器注入**

默认加在IOC容器中的组件，容器启动会调用**无参**构造器创建对象，再进行初始化赋值操作。

如果在组件中只有一个有参构造器，这个有参构造器的`@Autowired`可以省略，参数位置的组件还是可以从IOC容器中获取。
```java
@Component
public calss  Boss {
    private Car car;

	// @Autowired  // 构造器使用的参数，自定义类型的值从IOC容器中获取
    public Boss(Car car) {
        this.car = car;  // IOC容器中的car
    }
}
```
**参数注入**
```java
@Component
public calss  Boss {
    private Car car;
    public Boss(@Autowired Car car) { this.car = car; }
    // public setCar(@Autowired Car car) { this.car = car; }
}
```
**创建时注入**

`@Bean`标注的方法创建对象时，方法参数的值从容器中获取
```java
@Configurationpublic class MainConfig {
    @Bean
    public Boss boss(@Autowired Car car) {  // @Autowired可以省略
        Boss boss = new Boss();
        boss.setCar(car);  // IOC容器中的car
        return boss;
    }
}
```
```java
public class Boss {
    private Car car;
	public void setCar(Car car) { this.car = car; }
}
```
### 底层注入

如果想把Spring容器底层的一些组件注入到自定义组件中（`ApplicationContext`、`BeanFactory`等），可以使自定义组件实现`xxxAware`，则在创建对象时会调用接口规定的方法注入相关组件。`xxxAware`功能使用`xxxAwareProcessor`后置处理器实现。
```java
public class Car implements ApplicatinContextAware, BeanNameAware {
    private ApplicationContext ctx;

    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        this.ctx = ctx;
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("当前bean的名字： " + name);
    }
}
```
### 动态注入

`@Profile`：根据当前环境（如D、Q、P），动态激活和切换一系列组件的功能。可以放在 方法、类 上。

**设置组件**

- 加了环境标识的bean，只有这个环境被激活的时候才注册到容器中。默认是default环境。

- 写在配置类上，只有在指定环境时，整个配置类里面的所有配置才开始加载；

- 没有标注环境表示的bean，在任何环境下都加载
```java
PropertySource("classpath:/db.properties")
@Configuration
// @Profile("test") 为整个类设置环境
public class MainConfig implements EmbeddedValueResolverAware {

    @Value("${db.usr}")
    private String usr;

    private String driver;

    @Profile("default")  // 默认环境
    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.pwd}") String pwd) {
        ComboPooledDataSource data = new ComboPooledDataSource();
        data.setyUser(usr);
        data.setPassword(pwd);
        data.setDriverClass(driver);
        data.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        return data;
    }

    @Profile("dev")
    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${db.pwd}") String pwd) {
        ComboPooledDataSource data = new ComboPooledDataSource();
        data.setyUser(usr);
        data.setPassword(pwd);
        data.setDriverClass(driver);
        data.setJdbcUrl("jdbc:mysql://localhost:3306/dev");
        return data;
    }

    @Profile("prod")
    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${db.pwd}") String pwd) {
        ComboPooledDataSource data = new ComboPooledDataSource();
        data.setyUser(usr);
        data.setPassword(pwd);
        data.setDriverClass(driver);
        data.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        return data;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.driver = resolver.resolverStringValue("${db.driver}");
    }
}
```
**激活环境**

- 命令行参数：在虚拟机参数位置加载`Dspring.profiles.active=test`；

- 代码：`setActiveProfiles`方法
```java
public class MainTest {
    public static void main(Strings[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext();
        ctx.getEnvironment().setActiveProfiles("test", "dev");
        ctx.register(MainConfig.class);  // 注册配置类
        ctx.refresh();  // 刷新容器
    }
}
```
# AOP

AOP（动态代理）指在程序运行期间动态将某段代码切入到指定位置进行运行到编程方式。

步骤：

- 导入AOP模块，Spring AOP：`spring-aspects`；

- 定义业务逻辑类（Calculator），在业务逻辑运行的时候进行日志打印；

- 定义切面类（Log），切面类方法需动态感知`Calculator.div`运行；

- 给切面类目标方法标注何时运行；

- 将切面类和业务逻辑类加入到容器；

- 告诉Spring哪个类是切面类`@Aspect`；

- 配置类加上注解`@EnableAspectJAutoProxy`开启自动代理；

不要自己`new`对象，不然没效果，要从IOC容器中取才有效果。
```xml
<beans>
  <!-- 开启基于注解的切面功能 -->
  <aop:aspectj-autoproxy />
</beans>
```
```java
@EnableAspectJAutoProxy  // 开启基于注解的切面代理
@Configuration
public class MainConfig {
    @Bean
    public Calculator calculator() {...}

    @Bean
    public Log log() {...}
}
```
```java
@Aspect  // 切面类
public class Log {

    // 抽取公共切入点表达式
	@Pointcut("execution(public int com.demo.aop.Calculator.*(..))")  // 切入点表达式
    public void pointCut() {}

    // 前置通知，目标方法运行前运行
    @Before("public int com.demo.aop.Calculator.div(int, int)")
    public void logStart(JoinPoint joinPoint) {  // joinPoint必须为参数列表第一位
        Object[] args = joinPoint.getArgs();
        System.out.println("" + joinPoint.getSignature().getName() + "start...");
    }

    // 后置通知，目标方法结束（不论正常异常）后运行
    @After("public int com.demo.aop.Calculator.*(..)")
    public void logEnd() {
        System.out.println("");
    }

    // 返回通知，目标方法正常返回之后运行
    @AfterReturning(value="pointCut()", returning="result")  // 本类直接引用切入点表达式，外类要加全路径名
    public void logReturn(Object result) {
        System.out.println("result is " + result);
    }

    // 异常通知，目标方法出现异常之后运行
    @AfterThrowing(value="com.demo.aop.Log.pointCut()", throwing="exception")
    public void logException(Exception exception) {
        System.out.println("exception msg: " + exception);
    }

    // 环绕通知，动态代理，手动推动目标方法运行
    @Around
}
```
