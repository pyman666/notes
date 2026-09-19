
- [快速入门](#快速入门)
- [Bean容器](#bean容器)
- [生命周期](#生命周期)
- [装配Bean](#装配bean)
  - [xml装配](#xml装配)
  - [注解装配](#注解装配)
  - [特殊Bean](#特殊bean)
- [通知](#通知)

## 快速入门

## Bean容器

**BeanFactory**：bean工厂容器（不常用）

- 最简单的容器，工厂设计模式，提供了基础的DI支持，创建各种类型的Bean，参与bean的生命周期；

- bean工厂只把bean的定义信息加载进来，用到的时候才会实例化，节约内存速度慢，较少使用（如移动设备）；

**ApplicationContext**：应用上下文容器（常用）

- 更加高级的容器，建立在Bean工厂基础上；

- 提供文本信息解析工具，包括对国际化支持；

- 提供载入文件资源的通用方法，如图片；

- 可以向注册为监听器的bean发送事件；

- 配置的bean如果是**单例**，不管用不用都会被实例化，预加载提高速度占内存；

- 三种实现方式：

- `ClassPathXmlApplicationContext`：从类路径加载；

- `FIleSystemXmlApplicationContext`：从文件系统加载；

- `XmlWebApplicationContext`：从web系统加载
```xml
<beans xmlns="">
  <bean id="changeLetter" class="com.service.UpperLetter" scope="singleton">
    <property name="str" value="abc"/>
  </bean>
</beans>
```
```java
// 实例化容器时不会创建bean，当使用某个bean时才会实时创建
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("com/service/beans.xml"));
ChangeLetter change = (ChangeLetter) factory.getBean("changeLetter");
```
```java
// 实例化容器时就会创建bean（单例）
ApplicationContext ac = new ClassPathXmlApplicationContext("com/service/beans.xml");
ApplicationContext ac = new FIleSystemXmlApplicationContext("D:\\java\\src\\com\\service\\beans.xml");
ChangeLetter change = (ChangeLetter) ac.getBean("changeLetter");
```
## 生命周期

BeanFactory容器中bean的生命周期简洁一些，ApplicationContext容器相对更复杂。

单例bean被载入ApplicationContext容器（加载xml）时，他的生命周期就开始了：

1. 容器寻找bean的定义信息并实例化；

2. 使用DI按照bean定义信息配置bean的所有属性；

3. 若bean实现了`BeanNameAware`接口，工厂调用bean的`setBeanName`方法传递bean的id；

4. 若bean实现了`BeanFactoryAware`接口，工厂调用bean的`setBeanFactory`方法传入工厂自身；

5. 若bean实现了`ApplicationContextAware`接口，工厂调用bean的setApplicationContext方法传入上下文；

6. 若有`BeanPostProcessor`和bean关联，则他们的`postProcessBeforeInitialization`方法被调用；

7. 若bean实现了`InitializingBean`接口，`afterPropertiesSet`方法将被调用；

8. 若配置了自己的初始化方法`init-method`，或加了注解`@PostConstruct`，则其将被调用；

9. 若有`BeanPostProcessor`和bean关联，则他们的`postPorcessorAfterInitialization`方法被调用；

如此，bean就可以使用了；

当容器关闭时：

1. 若bean实现了`DisposableBean`接口，调用其`destroy`方法；

2. 若配置了自己的销毁方法`destroy-method`，或加了注解`@PreDestroy`，则其将被调用；
```xml
<beans xmlns="">
  <bean id="personService" class="com.service.PersonService" init-method="init" destroy-method="mydestroy">
    <property name="name" value="韩顺平"/>
  </bean>
  <bean id="myBeanPostProcessor" class="com.service.MyBeanPostProcessor"/>
</beans>
```
```java
public class PersonService implements BeanNameAware, BeanFactoryAware, ApplicationContextAware, DisposableBean {
    private String name;
    //getName()、setName...

    // 该方法可以用arg0表示正在被实例化的bean id
    public void setBeanName(String arg0) {
        System.out.println("bean name is :" + arg0);  // personService
    }

    // 该方法可以传递bean工厂
    public void setBeanFactory(BeanFactory arg0) {
        System.out.println("bean factory is :" + arg0);
    }

    public void setApplicationContext(BeanFactory arg0) {
        System.out.println("bean factory is :" + arg0);
    }

    public void afterPropertiesSet() {
        System.out.println("afterPropertiesSet");
    }

    public void init() {
        System.out.println("my init");
    }

    public void destroy() {
        // 可以关闭数据连接、socket、文件流、释放bean资源等；
        // 不建议使用
    }

    public void mydestroy() {
    }
}
```
```java
public class MyBeanPostProcessor implements BeanPostProcessor {
    public Object BeanPostProcessAfterInitialization(Object arg0, String arg1) {
        System.out.println("after...");
        return arg0;  // arg0其实就是personService
    }

    public Object BeanPostProcessBeforeInitialization(Object arg0, String arg1) {
        System.out.println("before...");
        return arg0;
    }
}
```
## 装配Bean

在Spring容器内拼凑Bean叫做装配。装配bean时，需要告诉容器哪些bean，以及容器如何使用DI将他们配合在一起。

### xml装配

xml时最常见的spring应用系统配置源，几种spring容器都支持xml装配bean，包括：

- `XmlBeanFactory`：调用`ClassPathResource`载入上下文定义文件；

- `ClassPathXmlApplicationContext`：从类路径载入上下文定义文件；

- `XmlWebApplicationContext`：从web应用上下文中载入定义文件；

**scope**

- `singleton`：单例（**默认**）；

- `prototype`：多实例，每一个bean都是全新（影响性能，不建议）；

- `request`：一次请求有效（web）；

- `session`：session级有效（web）；

- `global-session`：applicationContext一致（web）

**property**

- 通过类的`setXxx`方法注入属性；

- 通过类的构造器注入属性；

set注入的缺点时无法清晰表达哪些属性必须哪些可选，构造注入的优势时通过构造强依赖关系，不可能实例化不完全或无法使用的bean
```xml
<beans xmlns="">
  <bean id="department" class="com.hsp.Department">

    <!--普通属性-->
    <property name="name" value="财务部"/>

    <!--数组属性-->
    <property name="empName">
      <list>
        <value>小明</value>
        <value>小强</value>
      </list>
    </property>

    <!--集合属性-->
    <property name="empSet">
      <set>
        <ref bean="emp1"/>
        <ref bean="emp2"/>
      </set>
    </property>

    <!--字典属性-->
    <property name="empMap">
      <map>
        <entry key="01" value-ref="emp1"/>
        <entry key="02" value-ref="emp2"/>
      </map>
    </property>

    <!--内部bean-->
    <property name="emp">
      <bean class="com.hsp.Employee"/>
    </property>

    <!--property属性-->
    <property name="pp">
      <props>
        <prop key="aa">AA</p>
        <prop key="bb">BB</p>
      </props>
    </property>

    <!--空属性-->
    <property name="bb">
      <null/>
    </property>
  </bean>

  <!--构造器注入属性-->
  <bean id="emp" class="com.hsp.Employee">
    <constructor-arg index="0" type="java.lang.String" value="大明"/>
    <constructor-arg index="1" type="int" value="23"/>
  </bean>

  <bean id="emp1" class="com.hsp.Employee">
    <property name="name" value="韩顺平"/>
  </bean>
  <bean id="emp2" class="com.hsp.Employee">
    <property name="name" value="韩老师"/>
  </bean>
</beans>
```
**继承**
```xml
<beans xmlns="">
  <bean id="student" class="com.hsp.Student">
    <property name="name" value="韩顺平"/>
  </bean>
  <!--会继承student的name，独立配置则会替换-->
  <bean id="graduate" parent="student" class="com.hsp.Graduate">
    <property name="degree" value="学士"/>
  </bean>
</beans>
```
**自动装配** ★

自动装配bean属性值：

- `no`：不自动装配；

- `byName`：寻找和属性名相同的bean，找不到装不上；

- `byType`：寻找和属性类型相同的bean，找不到装不上，找到多个报错；

- `constructor`：寻找和bean构造参数一致的bean，找不到或找到多个报错；

- `autodetect`：`constructor`和`byType`选一种；

- `default`：默认，需在`<beans default-autowire=xxx>`指定，`default-autowire`默认`no`；

自动装配和手动装配可以混合使用
```xml
<beans xmlns="">
  <bean id="dog" class="com.hsp.Dog">
    <property name="name" value="大黄"/>
  </bean>

  <!--法1-->
  <bean id="master" class="com.hsp.Master">
    <property name="name" value="韩顺平"/>
    <property name="dog" ref="dog"/>
  </bean>

  <!--法2-->
  <bean id="master" class="com.hsp.Master" autowire="byName">
    <property name="name" value="韩顺平"/>
  </bean>
</beans>
```
### 注解装配

启用注解：`<context:annotation-config />`

该配置可以激活在类中探测到的各种注解，也可以选择为这些注解激活单独的后置处理器。

### 特殊Bean

Spring提供了一些特殊的bean供使用：

- `BeanPostProcessor`接口提供机会来修改bean；

- `PropertyPlaceholderConfigurer`分散配置，配置文件分成几个分散的配置文件，如数据源；
```xml
<beans xmlns="">
	<context:property-placeholder location="classpath:com/hsp/db1.properties,classpath:com/hsp/db2.properties" />

  <bean id="master" class="com.hsp.Master" autowire="byName">
    <property name="usr" value="${db1.usr}"/>
    <property name="pwd" value="${db2.pwd}"/>
  </bean>
</beans>
```
# AOP

AOP（Aspect Oriented Programming）面向切面编程，针对所有或一类对象编程。核心：在**不**增加代码的基础上**还**增加新的功能。AOP在开发框架本身使用较多，实际项目中不是很多。

- **切面**：aspect，要实现的交叉功能，室系统模块化的一个切面或领域（如日志的**功能**）；

- **通知**：通知中连接点插入到应用系统中，是切面的实际实现，通知系统新的行为（如实现日志的**代码**）；

- **连接点**：应用程序执行过程中插入切面的地点，可以是方法调用、异常抛出等，或者要修改的字段（静态）；

- **切入点**：定义了通知应该应用在哪些连接点，通知可以应用到AOP框架支持的任何连接点（动态）；

- **目标对象**：被通知的对象，既可以是自己编写的类，也可以是第三方类（Test1Service）；

- **代理对象**：通知应用到目标对象后建的对象，系统其它部分不用为代理对象改变（proxyFactoryBean）；

- **引入**：为类添加新方法和属性；

- **织入**：将切面应用到目标对象从而创建一个新代理对象的**过程**。织入发生在目标对象生命周期的多个点上：

- 编译期：前面在目标对象编译时织入，这需要一个特殊的编译器；

- 类装载期：切面在目标对象被载入JVM时织入，这需要一个特殊的类载入器；

- 运行期：切面在应用系统运行时织入；

通过代理对象实现AOP时，获取的proxyFactoryBean是一个动态代理对象，如果目标对象实现了若干接口，则spring使用JDK动态代理技术；若目标对象没有实现接口，则spring使用CGLIB技术。

## 通知

可以把通知看作拦截器、过滤器

|  |  |  |
| --- | --- | --- |
| **通知类型** | **接口** | **描述** |
| 环绕通知 | Methodinterceptor | 拦截对目标方法调用 |
| 前置通知 | MethodBeforeAdvice | 在目标方法调用前调用 |
| 后置通知 | AfterRetumingAdvice | 在目标方法调用后调用 |
| 异常通知 | ThrowsAdvice | 当目标方法报错时调用 |
| 引入通知 | NameMatchMethodPointcutAdvisor | 自定义切入点 |
需求：调用TestService的sayHello(Test1ServiceInterface)、sayBye(Test2ServiceInterface)方法时完成日志记录，其中前置通知只针对sayHello：
```java
public class MyMethodBeforeAdvice implements MethodBeforeAdvice {
    // method: 被调用方法名字
    // args: 给method传递的参数
    // target: 目标对象
    public void before(Method method, Object[] args, Object target) {
        System.out.println("before...");
    }
}

public class MyAfterReturningAdvice implements AfterReturningAdvice {
    // returnValue: method的返回值，如果有的话
    public void before(Object returnValue, Method method, Object[] args, Object target) {
        System.out.println("after...");
    }
}

public class MyMethodInterceptor implements MethodInterceptor {
    public Object invoke(MethodInvocation arg0) {
        System.out.println("invoke 1...");
        Object obj = arg0.proceed();
        System.out.println("invoke 2...");
        return obj;
    }
}

public class MyThrowAdvice implements ThrowAdvice {
    public void afterThrowing(Method method, Object[] args, Object target, Exception e) {
        System.out.println("throw..." + e.getMessage());
    }
}
```
```xml
<beans xmlns="">

  <!--被代理对象-->
  <bean id="testService" class="com.hsp.TestService">
    <property name="name" Value="韩顺平"/>
  </bean>

  <!--通知-->
  <bean id="myMethodBeforeAdvice" class="com.hsp.MyMethodBeforeAdvice"/>
  <bean id="myAfterReturningAdvice" class="com.hsp.MyAfterReturningAdvice"/>
  <bean id="myMethodInterceptor" class="com.hsp.MyMethodInterceptor"/>
  <bean id="myThrowAdvice" class="com.hsp.MyThrowAdvice"/>

  <!--自定义切入点-->
  <bean id="myMethodBeforeAdviceFilter" class="org.springframework.aop.support.NameMatchMethodPointcutAdvisor">
   <property name="advice" ref="myMethodBeforeAdvice"/>
   <property name="mappedNames">
     <list>
       <value>sayHello</value>  <!--可以用正则-->
     </list>
   </property>
  </bean>

  <!--代理对象-->
  <bean id="proxyFactoryBean" class="org.springframework.aop.framework.ProxyFactoryBean">

    <!--代理接口集-->
    <property name="proxyInterfaces">
      <list>
        <value>com.hsp.Test1ServiceInterface</value>
        <value>com.hsp.Test2ServiceInterface</value>
      </list>
    </property>

    <!--把通知织入代理对象-->
    <property name="interceptorNames">
      <list>
      	<value>myMethodBeforeAdviceFilter</value>  <!--不是myMethodBeforeAdvice-->
        <value>myAfterReturningAdvice</value>
        <value>myMethodInterceptor</value>
        <value>myThrowAdvice</value>
      </list>
    </property>

    <!--被代理对象-->
    <property name="target" ref="testService"/>
  </bean>
</beans>
```
```java
ApplicationContext ac = new ClassPathXmlApplicationContext("com/service/beans.xml");
// 错误 TestService ts = (TestService) ac.getBean("testService");
Test1ServiceInterface ts = (Test1ServiceInterface) ac.getBean("proxyFactoryBean");
ts.sayHello();
((Test1ServiceInterface) ts).sayBye();
```
