
- [基本结构](#基本结构)
- [POJO](#pojo)
- [对象关系映射文件](#对象关系映射文件)
- [主配置文件](#主配置文件)
- [[例] 快速入门](#例-快速入门)
- [Configuration](#configuration)
- [SessionFactory](#sessionfactory)
  - [openSession 🆚 getCurrentSession](#opensession-getcurrentsession)
- [Session ★](#session-)
  - [get 🆚 load](#get-load)
  - [懒加载](#懒加载)
  - [缓存](#缓存)
- [Transaction](#transaction)
- [Query](#query)
- [Criteria](#criteria)
- [[例] HibernateUtil](#例-hibernateutil)
- [基本使用](#基本使用)
- [分页](#分页)
- [参数绑定](#参数绑定)
- [单向多对一](#单向多对一)
- [单向一对多](#单向一对多)
- [双向一对多](#双向一对多)
- [主键一对一](#主键一对一)
- [外键一对一](#外键一对一)
- [多对多](#多对多)
- [级联](#级联)
- [维护关系](#维护关系)

## 基本结构

## POJO

在使用hibernate时，要求和数据库的表相互映射的那个java类就是一个POJO（plain ordinary java object 普通javaBeans）类，一般放在com.xxx.domain包下。叫POJO主要是为了和EJB（Enterprise Java Beans）区分，有时也称为Data对象。POJO类的特点：

- 有一个主键属性，用于唯一标识该对象（这就是为什么hibernate设计者建议要映射的表需要一个主键）；

- 有其他属性；

- 有对各个属性操作的`get`/`set`方法；

- 属性一般是`private`修饰；

- 一定有一个无参的构造函数（用于hibernate反射用）；

- 可序列化，可以唯一标识该对象和在网络和文件传输；
```java
public class Student implements Serializable {  // 要求可序列化
	private static final long serialVersionUID = 1L;
    private Integer id;
    private String name;
    private Integer age;

    public Integer getId() { return id; }
    public void setId(Integer id) { this.id = id; }
    // getName、setName、getAge、setAge ...
}
```
## 对象关系映射文件

对象关系映射文件，用于指定domain对象和表的映射关系。一般放在domain对象同一文件夹下，命名为`XXX.hbm.xml`。

domain对象中的属性，只有配置到对象关系映射文件，才会被hibernate管理（映射文件属性相对于domain只能少不能多）。

有些属性可以不配置：

- table：默认以类名小写作为表名；

- type：根据类的属性类型自动选择

常见主键生成策略：

- increment：由Hibernate从数据库中取出主键的最大值，以该值为基础，每次增量为1

- hilo：high low，高低位方式很常用，需要一张额外的表保存hi的值

- sequence：由数据库提供的sequence生成主键，需数据库支持

- identity：由数据库生成标识符，这个主键必须设置为自增长

- native：由hibernate根据使用的数据库自行判断采用identity、hilo、sequence其中一种，灵活性很强

- uuid：Universally Unique Identifier，它保证同一时空所有机器都唯一

- foreign：使用另一个相关联对象的主键作为该对象主键，主要用于一对一关系中
```xml
<hibernate-mapping package="com.hn.domain">
  <class name="Student" table="student">
    <!-- id标签用于指定主键 -->
    <id name="id" type="java.lang.Integer">
      <!--主键生成策略-->
    	<generator class="sequence"/>
      <param name="sequence">student_seq</param>
    </id>
    <!-- 属性映射 -->
    <property name="name" type="java.lang.String">
      <column name="name" not-null="true"/>
    </property>
    <property name="age" column="age"/>
  </class>
</hibernate-mapping>
```
## 主配置文件

主配置文件一般叫`hibernate.cfg.xml`，用于配置数据库类型、Driver、用户名、密码等；

数据生成方式`hbm2ddl.auto`有四个属性，再sessionFactory建立的时候自动检查数据库表结构，或者将数据库schema的DDL导到数据库中。项目发布后最好只配置一次让数据库生成，然后取消配置。

- `create`：每次都会删除并重新创建；

- `update`：没有表则创建，表已存在则看配置文件是否改变，变了则更新。如增加一个属性，相当于在数据库中加了一个字段，那么update会自动在数据库中加上该字段；

- `create-drop`：在显式关闭SessionFactory时，将drop掉数据库中的schema；

- `validate`：每次插入数据前都会验证数据库中的表结构和hbm文件的结构是否一致
```xml
<hibernate-configuration>
  <session-factory>
    <!-- 初始化JDBC连接 -->
    <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
    <property name="connection.url">jdbc:mysql:///hibernate5</property>
    <property name="connection.username">root</property>
    <property name="connection.password">root</property>
    <!-- dialect方言，明确告诉hibernate连接是哪种数据库 -->
    <property name="dialect">org.hibernate.dialect.MySQL57Dialect</property>
    <!-- 数据库生成方式 -->
    <property name="hbm2ddl.auto">update</property>
    <!-- 关联对象配置文件 -->
    <mapping resource="com/hn/domain/Student.hbm.xm1"/>
  </session-factory>
</hibernate-configuration>
```
## [例] *快速入门*
```java
// 初始化注册服务对象
final StandardServiceRegistry registry = new StandardServiceRegistryBuilder().configure().build();  // 默认加载hibernate.cfg.xml配置文件，如果不同可在configure方法中指定
// 从元信息获取Session工厂
SessionFactory sessionFactory = new MetadataSources(registry).buildMetadata().buildSessionFactory();
//从工厂创建Session连接
Session session = sessionFactory.openSession();
// 开启事务
Transaction transaction = session.beginTransaction();

// 创建实例
Student student = new Student();
student.setName("zhangsan");
student.setAge(18);

session.save(student);
// 提交事务
transaction.commit();  // 强制要求事务提交
// 关闭Session
session.close();
```
# 几大接口

一开始有五大接口，但现在Configuration接口不怎么用了。

## Configuration

注：5版本以后Configuration使用较少。

Configuration负责Hibernate配置工作，创建SessionFactory对象。在Hibernate启动过程中，Configuration类的实例首先定位映射文件位置，读取配置，然后创建SessionFactory对象。
```java
Configuration cfg1 = new Configuration();  // 读取src下hibernate.properties，不推荐
Configuration cfg2 = new Configuration().configure();  // 读取src下hibernate.cfg.xml，不推荐
Configuration cfg2 = new Configuration().configure("hb.cfg.xml");  // 自己指定配置文件
```
## SessionFactory

SessionFactory接口负责初始化Hibernate，它充当数据存储源的代理，使用工厂模式创建Session对象；

SessionFactory可以缓存sql语句和数据（session级缓存）；

SessionFactory是**重量级**的（占资源，一般用**单例**），通常一个数据库一个SessionFactory。多个数据库时，可以为每个数据库指定一个SessionFactory。
```jsx
// 从元信息获取Session工厂
SessionFactory factory = new MetadataSources(registry).buildMetadata().buildSessionFactory();

//从工厂创建Session连接
Session session1 = factory.openSession();  // 获取新的session
Session session2 = factory.getCurrentSession();  // 获取和当前线程绑定的session，换言之，一个线程中的session是同一个，利于事务控制

// 显式关闭factory
factory.close();
```
### openSession 🆚 getCurrentSession

- `getCurrentSession`创建的session会绑定到当前线程中，`openSession`创建的不会；

- `getCurrentSession`创建的session在commit或rollback时会自动关闭，`openSession`创建的必须手动关闭；

- `getCurrentSession`创建的session进行查询需要事务提交，`openSession`创建的不需要；

- 使用`getCurrentSession`需要在hibernate.cfg.xml中加入：
```xml
<!--本地事务（JDBC事务，针对一个数据库的事务）-->
<property name="hibernate.current_session_context_class">thread</property>

<!--全局事务（JTA事务，事务涉及到多个数据库）-->
<property name="hibernate.current_session_context_class">jta</property>
```
## Session ★

Session接口是用来操作数据库的，是Hibernate开发的最重要接口。一个Session实例代表与数据库的一次操作（当然一次操作可以是CRUD组合）；

Hibernate中，Session是一个**轻量级**的接口，创建和销毁都不会占用太多资源。但是Session是非线程安全的，最好一个线程只创建一个Session。

Session可以看作介于数据连接与事务管理一种中间接口，可以将其想象成一个持久对象的缓冲区，Hibernate能检测到这些持久化对象的改变，并且及时刷新数据库。

有时也称Session是一个持久化管理器，因为Session负责执行被持久化对象的增删改查操作，类似于JDBC的Connection和Statement，诸如存储持久化对象至数据库，以及从数据库获取它们。

另外，Hibernate的Session不同于JSP的HttpSession。
```java
Transaction tx = session.beginTransaction();

// 增
Student student0 = new Student();
student0.setId(0);
student0.setName("韩顺平");
student0.setAge(28);
session.save(student0);

// 查，通过主键id获取对象实例
Student student1 = (Student) session.get(Student.class, 1);  // 立即加载
Student student2 = (Student) session.load(Student.class, 2);  // 懒加载

// 改，不先查询直接修改会使其他字段变成null
student1.setName("Jack");  // 产生update语句
student1.setName(18);  // 只会产生一个update语句

// 删
session.delete(student2);

tx.commit();
session.evict(student);  // 清除某个缓存
session.clear();  // 清除缓存
session.close();
```
### get 🆚 load

返回：

- `get`直接返回实体类，查不到返回null；

- `load`返回一个实体代理对象（不马上发sql，使用时才发），当代理对象被调用时，若数据不存在会抛异常；

加载：

- `get`先到缓存查，没有就马上发出sql到DB查；

- `load`先到缓存查，没有则返回一个代理对象（不马上到DB中查），等后面使用这个代理对象操作时，才到DB查（懒加载）；

总之，如果确定DB中有这个对象就用`load`，不确定就用`get`，从而提高效率；

### 懒加载

懒加载（Load On Demand）是指程序推迟访问数据库，例如查询一个对象时，默认只返回对象的普通属性（name）。当用户去使用对象属性时（grade），才会向数据库发出再一次查询；但如果此时session已关闭，对象属性查询就会失败。

Domain非final才能实现懒加载，懒加载可以保证有时候不必要的访问数据库。取消懒加载的方法：

- **法一**：在对象映射文件中配置`lazy="false"`。在many方（student）设置时，把它相关联的对象（grade）也查询，对select语句查询影响不大；在one方（grade）设置，不管用不用都会把所有关联的对象（student）也查询，很多查询不必要
```xml
<class name="Student" table="student" lazy="false">
</class>
```
- **法二**：明确初始化，在session还没关闭时getXxx()强制访问数据库，或者`Hibernate.initialize(代理对象);`

- **法三**：过滤器`openinSessionView`，相当于延长session周期
```java
public class MyFilter extends HttpServlet implements Filter {
    public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2) throw Exception {
        Session s = null;
        Transaction tx = null;
        try {
            s = HibernateUtil.getCurrentSession();
            tx = s.beginTransaction();
            arg2.doFilter(arg0, arg1);
            tx.commit();
        } catch (Exception e) {
            if (tx != null) {
                tx.rollback();
            }
            throw new RuntimeException(e.getMessage());
        } finally {
            HibernateUtil.closeSession();
        }
    }
}
```
- **法四**：在ssh中，可以实现service层，标注方式解决懒加载

### 缓存

**一级缓存**：Session级共享。

- 查询对象时，首先到一级缓存中查。有就不到数据库中取，没有到db取，同时在一级缓存中放入对象；

- `save`、`update`、`saveOrUpdate`、`load`、`get`、`list`、`iterate`、`lock`等方法都会将对象放在一级缓存中；

- `get`、`load`会首先从一级缓存中取数据，`query.list`、`query.uniqueResult`不会从一级缓存取；

- 一级缓存不需要配置就可以使用，本身没有保护机制，不能控制缓存的数量，所以要注意大批量操作数据时可能造成内存溢出，可用`evict`（清一个）、`clear`（清所有）等方法清除缓存中的内容；

- session关闭后，一级缓存中的对象会自动销毁

**二级缓存**：SessionFactory级共享。

- 默认没有，交给第三方，常见的有OSCache、EHCache；

- 二级缓存策略：只读`read-only`、读写`read-write`（推荐，如银行项目）、不严格读写`nonstrict-read-write`（如帖子浏览次数）、事务缓存`transactional`（不常用）；
```xml
<session-factory>
  <property name="cache.use_second_level_cache">true</property>
  <property name="cache.provider_class">org.hibernate.cache.OSCacheProvider</property>
  <property name="cache.hibernate.generate_statistics">true</property>
  <!-- 哪个对象启用缓存 -->
  <class-cache class="com.hn.domain.Student" usage="read-write"/>
</session-factory>
```
## Transaction

Transaction是Hibernate事务接口，负责事务相关操作，本质上也是数据库事务。Hibernate要求显式地调用事务（查询可以不调用），但是使用Hibernate一般使用Spring去管理事务。
```java
// 法一：创建对象并开启事务
Transaction tx1 = session.beginTransaciton();
// 法二：创建对象，用begin开启事务
Transaction tx2 = session.getTransaciton();
tx2.begin();

tx.commit();  // 提交事务，hibernate强制要求事务提交
tx.rollback();  // 回滚事务
```
## Query

Query负责执行各种数据查询功能，可以完成更加复杂的查询任务，支持Hibernate特有的HQL语句和SQL语句。
```java
Query query = session.createQuery("from Student where id=10");  // from domain而不是表名，id可以是domain属性（推荐），也可以表column
List<Student> students = query.list();  // 会自动封装成对应的domain对象
```
## Criteria

Criteria接口是比HQL更加面向对象的查询方式
```java
List<Student> students = session.createCriteria(Student.class)
    .add( Restrictions.like("name", "Jac%") )
    .add( Restrictions.gt("age", 18) )
    .addOrder( Order.asc("id") )
    .setMaxResults(50)
    .list();
```
## [例] *HibernateUtil*
```java
// 线程局部模式
private static ThreadLocal<Session> threadLocal = new ThreadLocal<>();
private static StandardServiceRegistry registry;
private static SessionFactory factory;

static {  // 静态块初始化Session工厂
    registry = new StandardServiceRegistryBuilder().configure().build();
    factory = new MetadataSources(registry).buildMetadata().buildSessionFactory();
}

public static Session openSession() {
    return factory.openSession();
}

public static Session getSession() {
    Session session = threadLocal.get();  // 从线程获取session
    if (session == null || !session.isOpen()) {
        session = factory.openSession();
        threadLocal.set(session);  // 把session设置到threadLocal，相当于session和线程绑定
    }
    return session;
}

public static void closeSession() {
    Session session = threadLocal.get();
    threadLocal.set(null);
    if (null != session && session.isOpen()) {
    	session.close();
    }
}

public List executeQuery(String hql, String[] params) {
    Session session = null;
    List list = null;
    try {
        session = openSession();
        Query query = session.createQuery(hql);
        if ( params != null && params.length>0 ) {
            for ( int i=0; i<params.length; i++ ) {
                query.setString(i, params[i]);
            }
        }
        list = query.list();
    } catch ( Exception e ) {
        e.printStackTrace();
        throw new RuntimeException(e.getMessage);
    } finally {
        if( session != null && session.isOpen() ) {
            session.close();
        }
    }
    return list;
}

public List executeQueryByPage(String hql, String[] params, int pageSize, int pageNow) {
    Session session = null;
    List list = null;
    try {
        session = openSession();
        Query query = session.createQuery(hql);
        if ( params != null && params.length>0 ) {
            for ( int i=0; i<params.length; i++ ) {
                query.setString(i, params[i]);
            }
        }
        query.setFirstResult( (pageNow-1)*pageSize ).setMaxResult(pageSize);
        list = query.list();
    } catch ( Exception e ) {
        e.printStackTrace();
        throw new RuntimeException(e.getMessage);
    } finally {
        if( session != null && session.isOpen() ) {
            session.close();
        }
    }
    return list;
}

public static void save(Object obj) {
    Session s = null;
	Transaction tx = null;
	try {
        s = openSession();
        tx = s.beginTransaction();
        s.save(obj);
        tx.commit();
    } catch( Exception e ) {
        if (tx != null) {
            tx.rollback();
        }
        throw new RuntimeException(e.getMessage());
    } finally {
        if (s!=null && s.isOpen()) {
            s.close();
        }
    }
}

public static void executeUpdate(String hql, String[] params) {
    Session s = null;
    Transaction tx = null;
    try {
        s = openSession();
        tx = s.beginTransaction();
        Query query = s.createQuery(hql);
        if ( params != null && params.length>0 ) {
            for ( int i=0; i<params.length; i++ ) {
                query.setString(i, params[i]);
            }
        }
        int row = query.executeUpdate();
        tx.commit();
    } catch ( Exception e ) {
        if (tx != null) {
            tx.rollback();
        }
        e.printStackTrace();
        throw new RuntimeException(e.getMessage);
    } finally {
        if( s != null && s.isOpen() ) {
            s.close();
        }
    }
}
```
# HQL

HQL（Hibernate query language）是面向对象的查询语句，主要通过Query来操作；与SQL不同，HQL中对象名区分大小写（除了java类和属性其它部分不区分大小写）；HQL查询的是对象而不是表，且支持多态

## 基本使用
```java
// 查询所有学生的所有属性，list
Query query = session.createQuery("from Student");
List<Student> students = query.list();  // 可以转为Student

// 部分属性，Object[]
Query query = session.createQuery("select name, age from Student");
List list = query.list();  // 不能直接转为Student
for ( int i=0; i<list.size(); i++ ) {
    Object[] objs = (Object[]) list.get(i);
    System.out.println( "name: " + objs[0].toString() + ", age: " + objs[1].toString() );
}

// 单个属性，Object
Query query = session.createQuery("select name from Student");
List list = query.list();  // 不能直接转为Student
for ( int i=0; i<list.size(); i++ ) {
    Object objs = (Object) list.get(i);
    System.out.println( "name: " + objs[0].toString() );
}

// 单条记录，uniqueResult效率较高
Query query = session.createQuery("from Student where id=10");
Student students = query.uniqueResult();  // 没有或只有一个可以直接返回，多个报错
```
常用方法：
```java
"select distinct name, age from Student";
"select age from Student where age between 18 and 22";
"from Student where name in ('Tom', 'Jack')";
"select avg(age), gender from Student group by gender";
"select count(*), gender from Student group by gender having count(*)>3";
```
多表查询：
```java
"select student.name, course.name, grade from Grade where grade<60";  // 三表联查

List<Grade> list = HibernateUtil.executeQuery("from Grade where course.id=21", null);
for (Grade g: list) {
    g.getGrade();  // 可以
    // g.getStudent().getName();  // 报错，懒加载session已关闭
}
```
## 分页
```java
private static void showResultByPage(int pageSize) {
    int pageNow = 1;
    int pageCount = 1;
    int rowCount = 1;

    Session session = null;
    Transaction tx = null;
    try {
        session = HibernateUtil.getCurrentSession();
        tx = session.beginTransaction();

        rowCount = Integer.parseInt(session.createQuery("select count(*) from Student"));
        pageCount = (rowCount - 1) / pageSize + 1;
        for (int i=1; i<=pageCount; i++) {
            System.out.println("--------第" + i + "页--------");
            List<Student> list = session.createQuery("from Student order by age")
                .setFirstResult( (i-1)*pageSize )  // 从第几条取，默认0
                .setMaxResult( pageSize )  // 取出几条，默认取所有
                .list();
            for (Student s: list) {
                System.out.println(s.getName() + ": " + s.getAge());
            }
        }
        tx.commit();
    } catch (Exception e) {
        e.printStackTrace();
        if ( tx != null ) {
            tx.rollback();
        }
        throw new RuntimeException(e.getMessage());
    } finally {
        if ( session != null && session.isOpen() ) {
            session.close();
        }
    }
}
```
## 参数绑定

参数绑定优点：可读性好、性能提高、防止sql注入
```java
// 法一
Query query = session.createQuery("from Student where name=:name and age>:age");
query.setString("name", "Tom");
query.setString("age", "18");  // 冒号按名称

// 法二
Query query = session.createQuery("from Student where name=? and age>?");
query.setString(0, "Tom");
query.setString(1, "18");  // 问号按位置
```
# 生命周期

Hibernate中对象有三种状态：

- **瞬时**（Transient）：数据库中没有数据对应，超过作用域会被JVM回收，一般是刚用`new`语句创建且与session没有关联的对象；

- **持久**（Persistent）：数据库中有数据对应，当前有session关联，并且相关联的session没有关闭，事务没有提交。持久对象发生改变，在事务提交时**会影响到数据库**（hibernate能检测到）；

- **游离**（Detached）：数据库中有数据对应，当前没有session关联。游离对象改变，hibernate不能检测到。

|  |  |  |  |
| --- | --- | --- | --- |
|  | **session** | **数据库** | **内存** |
| **瞬时** | × | × | √ |
| **持久** | √ | √ | √ |
| **游离** | × | √ | √ |
各状态转换：
```java
Session session = null;
Transaction tx = null;
User user = null;

try {
    session = HibernateUtil.getCurrentSession();
    tx = session.beginTransaction();
    // 瞬时状态
    student = new Student();
    student.setName("Jack");
	// 持久状态（处于session管理，且被保存数据库中）
    session.save(student);
    // 持久状态
    student = session.get(Student.class, 1);
    student.setName("Tom");
    session.update(student);
    // 游离状态
    session.evict(student);  // 清除某个缓存
    session.clear();  // 清除缓存
    // 临时状态
    session.delete(student);
    tx.commit();
} catch (Exception e) {
    e.printStackTrace();
    tx.rollback();
} finally {
    // 游离状态
    HibernateUtil.closeSession();
}
```
# 关系映射

映射关系是将数据库中的表映射成与之对应的对象，当对对象进行操作时，Hibernate会对数据库中的表执行相应的操作。

- 一对一：身份证🆚人

- 一对多：部门🆚员工

- 多对一：员工🆚部门

- 多对多：学生🆚老师（不建议）

## 单向多对一

如多个学生（many）在一个年级（one），student表的grade\_id列对应grade表的id
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="Student">
    <id name="id">
      <generator class="native"/>
    </id>
    <property name="name"/>
    <property name="age"/>
    <!-- 单向多对一
      name Student类属性名
      column student表列名
      class 对应的类，写了package可以不写全类名
      foreign-key 外键名，可以不写，默认随机
    -->
    <many-to-one name="grade" column="grade_id" class="Grade" not-null="true" "foreign-key"="fk_grade"/>
  </class>
</hibernate-mapping>
```
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="Grade">
    <id name="id">
      <generator class="native"/>
    </id>
    <property name="name"/>
  </class>
</hibernate-mapping>
```
```java
Grade grade1 = new Grage();
grade1.setName("低年级");
Grade grade2 = new Grage();
grade2.setName("高年级");

session.save(grade1);  // not-null=True，必须先save外键的一端
session.save(grade2);

Student student1 = new Student();
student1.setName("Tom");
student1.setAge(18);
student1.setGrade(grade1);
Student student2 = new Student();
student2.setName("Jack");
student2.setAge(15);
student2.setGrade(grade2);

session.save(student1);  // 没有update，性能高于单向一对多
session.save(student2);
```
## 单向一对多

Grade类多了集合属性students，能从grade表查询student
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="Student">
    <id name="id">
      <generator class="native"/>
    </id>
    <property name="name"/>
    <property name="age"/>
  </class>
</hibernate-mapping>
```
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="Grade">
    <id name="id">
      <generator class="native"/>
    </id>
    <property name="name"/>
    <!-- 单向一对多
      set Grade类中的集合属性
      name 集合属性的名称
      column 外键列名
      key 外键
      foreign-key 外键名
      one-to-many Grade类中属性students所表示的类型
    -->
    <set name="students">
      <key foreign-key="fk_grade" column="grade_id" not-null="true"></key>
      <one-to-many class="Student"/>
    </set>
  </class>
</hibernate-mapping>
```
```java
Student student1 = new Student();
student1.setName("Tom");
student1.setAge(18);
Student student2 = new Student();
student2.setName("Jack");
student2.setAge(15);

Grade grade1 = new Grage();
grade1.setName("低年级");
grade1.getStudents().add(student1);

Grade grade2 = new Grage();
grade2.setName("高年级");
Set<Student> set2 = new HashSet<>();
set2.add(student2);
grade2.setStudents(set2);

session.save(grade1);  // not-null=True，先save外键的一端
session.save(grade2);
session.save(student1);  // 有update，效率不如多对一
session.save(student2);
```
## 双向一对多

双向一对多关系映射中：

- 无须指定`not-null`；

- 关系由多的一端来维护，DML语句会少执行update语句，效率较高
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="Student">
    <id name="id">
      <generator class="native"/>
    </id>
    <property name="name"/>
    <property name="age"/>
    <!--双向不要not-null-->
    <many-to-one name="grade" column="grade_id" class="Grade" "foreign-key"="fk_grade"/>
  </class>
</hibernate-mapping>
```
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="Grade">
    <id name="id">
      <generator class="native"/>
    </id>
    <property name="name"/>
    <set name="students">
      <!--双向不要not-null-->
      <key foreign-key="fk_grade" column="grade_id"></key>
      <one-to-many class="Student"/>
    </set>
  </class>
</hibernate-mapping>
```
## 主键一对一

如人与身份证
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="Person" table="person">
    <id name="id" type="java.lang.Integer">
      <!--手动分配id-->
      <generator class="assigned"/>
    </id>
    <property name="name">
    	<column name="name" length="128"/>
    </property>
	<one-to-one name="idCard">
  </class>
</hibernate-mapping>
```
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="IdCard" table="idCard">
    <id name="id">
      <!--idCard的主键就是person的外键-->
      <generator class="foreign">
        <!--这里的值，指定跟哪个属性一对一-->
        <param name="property">person</param>
      </generator>
    </id>
    <!--没有constrained=true将不会生成外键约束-->
    <one-to-one name="person" constrained="true"/>
  </class>
</hibernate-mapping>
```
```java
Person p1 = new Person();
p1.setId("1234");
p1.setName("Jack");

IdCard c1 = new IdCard();
c1.setPerson(p1);

session.save(p1);  // 先有人
session.save(c1);
```
## 外键一对一

基于外键的一对一，可以描述为多对一，加unique约束，唯一的多对一，其实就是一对一了。
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="Person" table="person">
    <id name="id" type="java.lang.Integer">
      <!--手动分配id-->
      <generator class="assigned"/>
    </id>
    <property name="name">
    	<column name="name" length="128"/>
    </property>
	<one-to-one name="idCard">
  </class>
</hibernate-mapping>
```
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="IdCard" table="idCard">
    <id name="id">
      <generator class="assigned"/>
    </id>
    <!--唯一多对一，其实就是一对一，会在idCard表生成外键-->
    <many-to-one name="person" unique="true"/>
  </class>
</hibernate-mapping>
```
```java
Person p1 = new Person();
p1.setId("1234");
p1.setName("Jack");

IdCard c1 = new IdCard();
c1.setId("2234");
c1.setPerson(p1);

session.save(p1);  // 先有人
session.save(c1);
```
## 多对多

多对多在操作和性能方面都不太理想，所以使用较少，实际使用中最好转成两个一对多或多对一，这样程序好控制，同时不会出现冗余数据。

Hibernate会为我们创建中间关联表，转成两个一对多。
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="Student">
    <id name="id">
      <generator class="sequence">
      	<param name="sequence">stu_seq</param>
      </generator>
    </id>
    <property name="name">
    	<column name="name"/>
    </property>
    <!--一个学生对应多个选课-->
    <set name="stuCourses">
      <key column="student_id"></key>
      <one-to-many class="StuCourse"/>
    </set>
  </class>
</hibernate-mapping>
```
```java
<hibernate-mapping package="com.test.pojo">
  <class name="Course">
    <id name="id">
      <generator class="sequence">
        <param name="sequence">course_seq</param>
      </generator>
    </id>
    <property name="name">
      <column name="name"></column>
    </property>
    <!--一门课程对应多个选课记录-->
    <set name="stuCourses">
      <key column="course_id"></key>
      <one-to-many class="StuCourse"/>
    </set>
  </class>
</hibernate-mapping>
```
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="StuCourse">
    <id name="id">
      <generator class="sequence">
        <param name="sequence">stucourse_seq</param>
      </generator>
    </id>
    <property name="grade">
      <column name="grade"/>
    </property>
    <many-to-one name="course" column="course_id"/>
    <many-to-one name="student" column="student_id"/>
  </class>
</hibernate-mapping>
```
```java
Student s1 = new Student();
s1.setName("Jack");

Course c1 = new Course();
c1.setName("Java");

StuCourse sc = new StuCourse();
sc.setStudent(s1);
sc.setCourse(c1);

session.save(s1);  // 先有人
session.save(c1);
session.save(sc);
```
## 级联

级联操作cascade，就是在每次保存Student、Grade对象时，不需要都持久化至session，持久化一端另一端被自动持久化。

常用：none、all、save-update、delete、lock、refresh、evict、replicate、persist、merge、delete-orphan（one-to-many）。

- 集合属性、普通属性都能使用级联；

- 一般对多对一、多对多不设置级联，在一对一、一对多中设置，一般设置在主对象或one的一方（如雇员🆚部门中的部门）。

- 在多对一关系中，多的一端不能操作级联为`delete`，一般在多的一端设为`save-update`；

- 在一对多关系中，如果一的一端设为`delete`相关配置时，多的一端不能指明外键为非空
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="Grade">
    <id name="id">
      <generator class="native"/>
    </id>
    <property name="name"/>
    <!--自动save Student-->
    <set name="students" cascade="save-update,delete">
      <key column="grade_id"/>
      <one-to-many class="Student"/>
    </set>
  </class>
</hibernate-mapping>
```
```java
session.save(grade1);
session.save(grade2);
// session.save(student1);  无须显式保存，会自动保存
// session.save(student2);
```
## 维护关系

维护关系inverse，在双向一对多中，维护关系方由多端维护时效率较高，因为DML会少执行update语句。为了维护效率，一般都是将关系维护方设置为多端。

inverse值是boolean值，该属性只能在一端设置。从而不管代码层面如何实现，强制由多方维护关系
```xml
<hibernate-mapping package="com.test.pojo">
  <class name="Grade">
    <id name="id">
      <generator class="native"/>
    </id>
    <property name="name"/>
    <!--由另一端（多端）维护关系-->
    <set name="students" cascade="save-update" inverse="true">
      <key foreign-key="fk_grade" column="grade_id" not-null="true"></key>
      <one-to-many class="Student"/>
    </set>
  </class>
</hibernate-mapping>
```
