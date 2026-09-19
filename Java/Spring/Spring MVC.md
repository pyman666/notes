
- [快速入门](#快速入门)
- [放行](#放行)
- [事务](#事务)
- [异常](#异常)

## 快速入门

## 放行

静态资源如HTML、CSS等不需要SpringMVC拦截处理

```java
@Configuration
@ComponentScan({"com.demo.controller", "com.demo.config"})  // 加上Support配置类
public class SpringMvcConfig {

}
```

```java
@Configuration
public class SpringMvcSupport extends WebMvcConfigurationSupport {
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/pages/**").addResourceLocations("/pages/");
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");
        registry.addResourceHandler("/js/**").addResourceLocations("/js/");
        registry.addResourceHandler("/plugins/**").addResourceLocations("/plugins/");
    }
}
```

# 响应

`@ResponseBody`：设置当前控制器返回值作为响应体，`HttpMessageConverter`接口帮转

```java
@Controller
public class MyController {

    // 响应页面
	@RequestMapping("/page")
    public String toPage() {
        return "index.jsp";
    }

    // 文本
    @RequestMapping("/text")
    @ResponseBody
    public String toText() {
        return "response text";
    }

    // Pojo
    @RequestMapping("pojo")
    @ResponseBody
    public User toPojo() {
        User user = new User();
        return user;
    }

}
```

# RESTful

按REST风格访问资源时使用**行为动作**区分对资源进行何种操作，根据REST风格对资源进行访问称为RESTful。

[http://localhost:8080/user/select?id=1](http://localhost:8080/user/select?id=1) --> [http://localhost:8080/users/1](http://localhost:8080/users/1)

```java
@ResponseBody@Controller
public class MyController {

    @ResquestMapping(value="/users", method=RequestMethod.POST)
    @ResponseBody
    public void save() {}

    @ResquestMapping(value="/users/{id}", method=RequestMethod.DELETE)
    @ResponseBody
    public void delete(@PathVariable int id) {}

    @ResquestMapping(value="/users", method=RequestMethod.PUT)
    @ResponseBody
    public void update() {}

    @ResquestMapping(value="/users/{id}", method=RequestMethod.GET)
    @ResponseBody
    public void select(@PathVariable int id) {}

}
```

优化：

`@RestController` = `@Controller` + `@ResponseBody`

```java
@RestController
@ResponseBody
public class MyController {

    @PostMapping
    public void save() {}

    @DeleteMapping("/{id}")
    public void delete(@PathVariable int id) {}

    @PutMapping
    public void update() {}

    @GetMapping("/{id}")
    public void select(@PathVariable int id) {}

}
```

# SSM

SSM即 Spring+SpringMVC+MyBatis 框架集。

- Spring
- SpringConfig
- SpringMVC
- SpringMvcConfig
- ServletConfig
- MyBatis
- MybatisConfig
- JdbcConfig
- jdbc.properties

```java
@Configuration
@ComponentScan("com.demo.service")
@PropertySource("jdbc.properties")
@Import({JdbcConfig.class, MyBatisConfig.clsss})
public class SpringConfig {

}
```

```java
// SpringMcvConfig的容器可以访问SpringConfig的容器，反之不行
// 不想父子容器可以与SpringConfig合并
```

```java
public class ServletConfig extends AbstractAnnotationConfigDispatcherServletInitializer {
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvcConfig.class};
    }
    protected Class<?>[] getServletMappings() {
        return new Class[]{"/"};
    }
}
```

```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/db_name
jdbc.username=root
jdbc.password=root
```

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

    @Bean
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }

}
```

```java
public class MyBatisConfig {

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        factoryBean.setTypeAliasesPackage("com.demo.domain");
        return factoryBean;
    }

    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
        msc.setBasePackage("com.demo.dao");
        return msc;
    }

}
```

## 事务

```java
public class JdbcConfig {

    // 平台事务管理器
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
        return transactionManager;
    }

}
```

```java
@EnableTransacionManagement  // 开启事务
public class SpringConfig{

}
```

```java
@Transactional  // 接口挂上事务
public interface UserService{

}
```

## 异常

```java
@RestControllerAdvice
public class ExceptionAdvice {

    @ExceptionHandler(ChaoJiYingException.class)
    public void doException(ChaoJiYingException e) {

    }
}
```

# 拦截器

拦截器（Interceptor）是一种动态拦截方法调用的机制，在SpringMVC中动态拦截控制器方法的执行。

拦截器与过滤器区别：

- 归属不同：Filter属于Servlet技术，Interceptor属于SpringMVC技术
- 拦截内容不同：Filter对所有访问进行增强，Interceptor仅针对SpringMVC的访问进行增强

![](https://cdn.nlark.com/yuque/0/2022/png/2341594/1669212698201-fb18c54f-15fc-4e53-9986-001adbed5a86.png)

```java
@Configuration
@ComponentScan({"com.demo.controller", "com.demo.config"})  // interceptor目录在controller目录下
public class SpringMvcConfig {

}
```

```java
@Configuration
public class SpringMvcSupport extends WebMvcConfigurationSupport {

    @Autowired
    private ProjectInterceptor interceptor;

    @Override  // 上文放行静态请求
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");
    }

    @Override  // 拦截请求
    protected void addInterceptors(ResourceHandlerRegistry registry) {
        registry.addInceptor(interceptor).addPathPatterns("/users", "/users/*");
    }
}
```

可以简化成：

```java
@Configuration
@ComponentScan("com.demo.controller")
public class SpringMvcConfig implements WebMvcConfigurer {

    @Autowired
    private ProjectInterceptor interceptor;

    @Override  // 上文放行静态请求
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/css/**").addResourceLocations("/css/");
    }

    @Override  // 拦截请求
    protected void addInterceptors(ResourceHandlerRegistry registry) {
        registry.addInceptor(interceptor).addPathPatterns("/users", "/users/*");
    }
}
```

拦截器功能类：

```java
// 拦截器功能类
@Component
public class ProjectInterceptor implements HandlerInterceptor {

    @Override  // 多个拦截器先放的先运行
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // request.getHeader("Content-Type")...
        // response.getCookie()...

        //HandlerMethod hm = (HandlerMethod) handler;  被调用的处理器对象，本质上是一个方法
        //hm.getMethod()...  对反射技术中的Method对象进行了再包装

        return true;  // false终止原始操作
    }

    @Overrid  // 先放后运行，有拦截器pre false了都不允许
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        // modelAndView 页面跳转相关操作
    }

    @Override  // 先放后运行，相应拦截器pre true了就运行
    public void afterHandle(HttpServletRequest request, HttpServletResponse response, Object handler, Exception e) throws Exception {
        // e controller表现层异常
    }

}
```
