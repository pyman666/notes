
- [连接数据库](#连接数据库)
- [SQL注入](#sql注入)
- [PrepareStatement](#preparestatement)
- [C3P0](#c3p0)
- [Druid](#druid)

## 连接数据库

## SQL注入

sql注入是利用某些系统没有对用户输入的数据进行充分的检查，而在用户输入数据中注入非法sql语句段或命令，恶意攻击数据库。要防范sql注入，只要用PreparedStatement取代Statement即可
```sql
-- 用户名为:  1' OR
-- 万能密码:  OR '1' = '1
SELECT * FROM admin WHERE NAME = '1' OR' AND pwd = 'OR '1' = '1'
```
## PrepareStatement

PreparedStatement执行的sql语句中的参数用问号来表示，调用预处理对象的setXxx()方法设置这些参数，第一个参数是要设置的sql语句中的参数的索引（从1开始），第二个是设置的sql语句中的参数的值

- 不再使用 + 拼接sql语句，减少语法错误；

- 有效解决了sql注入问题；

- 大大减少了编译次数，效率较高
```java
String sql = "select * from admin where name=? and pwd=?";
PreparedStatement ppStatement = connect.prepareStatement(sql);
ppStatement.setString(1, name);
ppStatement.setString(2, pwd);
bool result = ppStatement.execute();  // 执行任意sql，返回布尔
```
# 事务

JDBC程序中，当一个Connection对象创建时，默认情况下是自动提交事务：每次执行一个sql语句时，如果执行成功，就会向数据库自动提交，而不能回滚。为了让多个sql语句作为一个整体执行，需要使用事务：
```java
try {
    connection.setAutoCommit(false);  // 取消自动提交事务

    ppStatement1 = connection.prepareStatement(sql1);
    ppStatement1.executeUpdate();
    int i = 1 / 0;
    ppStatement2 = connection.prepareStatement(sql2);
    ppStatement2.executeUpdate();

    connection.commit();  // 提交事务

} catch (SQLException e) {
    connection.rollback();  // 默认回滚到事务开始的状态
}
```
# 批处理

当需要成批插入或者更新记录时，可以采用Java的批量更新机制，这一机制允许多条语句一次性提交给数据库批量处理，通常比单独提交处理更有效率。

批处理往往和PrepareStatement搭配使用，既可以减少编译次数，又可以减少运行次数，大大提高效率

JDBC连接mysql时，如果需要批处理，需要在url中添加参数`?rewriteBatchedStatements=true`
```java
String url = "jdbc:mysql://localhost:3306/my_db?rewriteBatchedStatements=true";

for (int 1 = 0; i < 5000; i++) {
    ppStatement.setString(1, "Jack");
    ppStatement.setString(1, i);

    ppStatement.addBatch();  // 添加需要批处理的sql或参数
    if( (i + 1) % 1000 == 0 ) {  // 满1000条批量执行
        ppStatement.executeBatch();  // 执行批处理语句
        ppStatement.clearBatch();  // 清空批处理包的语句
    }
}
```
# 连接池

传统JDBC数据库连接使用DriverManager来获取，每次向数据库建立连接的时候都要将Connection加载到内存中，再验证IP地址、用户名和密码（0.05~1s），需要数据库连接的时候就向数据库请求一个，频繁地进行数据库连接操作将占用很多的系统资源，容易造成服务器崩溃；每次数据库连接使用完后都得断开，如果程序出现异常而未能关闭，将导致数据库内存泄漏，最终导致重启数据库；传统获取连接的方式，不能控制创建的连接数量，如果连接过多，也可能导致内存泄漏，数据库崩溃。

数据库连接池（connection pool）预先在缓冲池中放入一定数量的连接，当需要建立数据库连接时，只需从缓冲池中去除一个，用完再放回去。

数据库连接池负责分配、管理和释放数据库连接，它允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个。

当应用程序像连接池中请求的连接数超过最大连接数量时，这些请求将被加入到等待队列中。

JDBC的数据库连接池使用javax.sql.DataSource来表示，DataSource只是一个接口，通常由第三方提供：

- C3P0连接池，速度相对较慢，稳定性不错（hibernate、spring）；

- DBCP连接池，熟读相对c3p0较快，但不稳定；

- Proxool连接池，有监控连接池状态的功能，稳定性较c3p0差一点；

- BoneCP连接池，速度快；

- Druid连接池，（德鲁伊）是阿里提供的，集多个连接池优点于一身

## C3P0

法一，通过参数
```java
import com.mchange.v2.c3p0.ComboPooledDataSource

// 创建数据源对象
ComboPooledDataSource dataSource = new ComboPooledDataSource();
// 连接由ComboPooledDataSource来管理
dataSource.setDriverClass(driver);
dataSource.setJdbcUrl(url);
dataSource.setUser(user);
dataSource.setPassword(password);
dataSource.setInitialPoolSize(10);  // 初始化连接数
dataSource.setMaxPoolSize(50);  // 最大连接数
// 获取连接
Connection connection = dataSource.getConnection();
connection.close();  // 连接池中，close不是断掉连接，而是把Connection放回连接池
```
法二，利用配置文件
```
将该文件考到src目录下并配置相关参数
```
```java
ComboPooledDataSource dataSource = new ComboPooledDataSource("数据源名称");  // 填入数据源名称
Connection connection = dataSource.getConnection();
connection.close();
```
## Druid
```java
// 首先将配置文件考到src目录下并配置相关参数
Properties properties = new Properties();
properties.load(new FileInputStream("src\\druid.properties");

DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);  // 填入数据源名称
Connection connection = dataSource.getConnection();
connection.close();  // 放回连接池
```
# DBUtils

问题：ResultSet和Connection关联，如果关闭连接，就不能使用结果集，不利于数据的管理（只能用一次），且结果集使用返回信息也不方便（getSting不如getName直观）。

建立一个Java类与数据表映射，这个类一般叫JavaBean或POJO或Domain，将ResultSet封装到`ArrayList<JavaBean>`中

commons-dbutils是Apache组织提供的一个开源JDBC工具类库，是对JDBC的封装，能极大简化JDBC编码的工作量。

- `DbUtils`类；

- `QueryRunner`类：封装了sql执行，线程安全，可实现增删改查、批处理；

- `ResultSetHandler`接口：处理java.sql.ResultSet，将数据按要求转换为另一种形式

- `ArrayHandler`：把结果集第一行转为数组

- `ArrayListHandler`：把结果集每一行转为数组存入List

- `BeanHandler`：把结果集第一行封装到对应POJO实例中

- `BeanListHandler`：每一行封装到POJO，存入List

- `ColumnListHandler`：把结果集某一列存入List

- `keyedHandler(name)`：每一行封装到Map，再把这些Map再存到一个Map，其key为指定key

- `MapHandler`：第一行封装到一个Map，key是列名，value是对应值

- `MapListHandler`：每一行封装到一个Map，存入List
```java
Connection conn = dataSource.getConnection();

// 引入DBUtils相关jar，加入项目
QueryRunner queryRunner = new QueryRunner();

// DQL
String sql = "select * from actor where id >= ?";
// 多行多列
List<Actor> list = queryRunner.query(conn, sql, new BeanListHandler<>(Actor.class), 1);  // 1是sql中?的赋值，是可变参数
for (Actor actor: list) {}
// 单行多列
Actor actor = queryRunner.query(conn, sql, new BeanHandler<>(Actor.class), 1);
if (actor != null) {}
// 单行单列
Object obj = queryRunner.query(conn, sql, new ScalarHandler<>(Actor.class), 1);
if (obj != null) {}

// DML
String sql = "update actor set name = ? where id = ?";
int row = queryRunner.update(conn, sql, "Tom", 1);  // DML都是update方法，返回受影响行数
if (row > 0) {}

conn.close();  // 底层得到的ResultSet、PrepareStatement会在query关闭
```
# BasicDAO

DBUtils+Druid简化了JDBC开发，但还有不足：

- sql语句固定，不能通过参数传入，通用性不好；

- 对于select操作，如果由返回值，返回类型不能固定，需要使用泛型；

- 随着业务逐渐复杂，不能只靠一个Java类完成

DAO（data access object）：数据访问对象（访问数据的对象）。

BasicDAO将各个DAO共同操作放一起，这样的通用类是专门和数据库交互的，完成对表的crud操作。可以简化代码，提高维护性和可读性；

在BasicDAO的基础上，实现一张表对应一个DAO，更好地完成功能
```java
public class BasicDAO<T> {
    private QueryRunner qr = new QueryRunner();

    // 开发通用DML方法，针对任意表
    public int update(String sql, Object... param) {
        Connection conn = null;
        try {
            conn = JDBCUtilsByDruid.getConnection();
            return qr.update(connection, sql, param);
        } finally {
        	conn.close();
        }
    }

    // 返回多个对象（多行），针对任意表
    public List<T> queryMulti(String sql, Class<T> clazz, Object... param){
        Connection conn = null;
        try {
            conn = JDBCUtilsByDruid.getConnection();
            return qr.query(connection, sql, new BeanListHandler<T>(clazz), param);
        } finally {
            conn.close();
        }
    }
}
```
```java
public class ActorDAO extends BasicDAO<Actor> {}
```
