
- [基本结构](#基本结构)
- [mapper接口](#mapper接口)
- [对象映射文件](#对象映射文件)
- [主配置文件](#主配置文件)
- [[例] 快速入门](#例-快速入门)
- [传入参数](#传入参数)
  - [特殊参数](#特殊参数)
- [获取结果](#获取结果)
  - [获取自增主键](#获取自增主键)
  - [字段名≠属性名](#字段名属性名)
- [动态SQL](#动态sql)
  - [if](#if)
  - [where](#where)
  - [trim](#trim)
  - [choose 🆚 when 🆚 otherwise](#choose-when-otherwise)
  - [foreach](#foreach)
  - [sql](#sql)
- [多对一](#多对一)
  - [懒加载](#懒加载)
- [一对多](#一对多)
- [一级缓存](#一级缓存)
- [二级缓存](#二级缓存)
- [查询顺序](#查询顺序)

## 基本结构

## mapper接口

MyBatis中的mapper接口相当于以前的DAO，区别在于mapper仅仅是接口，不需要提供实现类。

MyBatis面向接口编程，需保持：

- 对象映射文件的`namespace`和mapper接口的全类名一致；

- 对象映射文件的sql语句的`id`和mapper接口中的方法名一致；
```java
public interface UserMapper {
    int insertUser();
    void updateUser();
    void deleteUser();
    User getUserById();
    List<User> getAllUser();
}
```
```java
public class User {
    private Integer id;
    private String name;
    // set、get...
}
```
## 对象映射文件

以包为单位引入映射文件，要求：

- mapper接口（src）所在的包名和映射文件（resource）所在的包名一致；

- mapper接口名称要和映射文件名称一致；
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">

  <insert id="insertUser">
    insert into t_user values (null, 'admin')
  </insert>

  <update id="updateUser">
    update t_user set name = '尚硅谷' where id = 1
  </update>

  <delete id="deleteUser">
    delete form t_user where id = 1
  </delete>

  <!--查询功能须设置resultType（默认映射关系）或resultMap（自定义映射关系）-->
  <select id="getUserById" resultType="com.xxx.mybatis.pojo.User">
    select * from t_user where id = 1
  </select>

  <select id="getAllUser" resultType="User">  <!--类型别名-->
    select * from t_user
  </select>
</mapper>
```
## 主配置文件

一般放在resource文件夹下，也可以结合properties文件使用

标签顺序：`properties settings typeAliases typeHandlers objectFactory objectWrapperFactory reflectorFactory plugins environments databaseIdProvider mappers`
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mydb
jdbc.username=xxx
jdbc.password=xxx
```
```xml
<configuration>

  <!--引入properties文件-->
  <properties resource="jdbc.properties"/>

  <!--类型别名-->
  <typeAliases>
    <!--alias默认User，不区分大小写，使用较少-->
    <typeAliase type="com.xxx.mybatis.pojo.User" alias="User"/>
    <!--包下所有类型设置为默认别名，即类名，不区分大小写，使用较多-->
    <package name="com.xxx.mybatis.pojo"/>
  </typeAliases>

  <!--连接数据库的环境-->
	<environments default="development">
    <environment id="development">

      <!--事务处理器，type=JDBC（手动处理事务的提交或回滚）/MANAGED（被管理，如Spring）-->
      <transactionManager type="JDBC"/>

      <!--数据源，type=POOLED（用连接池缓存数据库连接）/UNPOOLED/JNDI（用上下文中的数据源）-->
			<dataSource type="POOLED">
        <property name="driver" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}/>
        <property name="password" value="${jdbc.password}"/>
      </dataSource>

    </environment>
  </environments>

  <!--引入映射文件-->
  <mappers>
    <!--逐个引入-->
    <mapper resource="com/xxx/mybatis/mappers/UserMapper.xml"/>
    <!--按包引入-->
    <package name="com.xxx.mybatis.mappers"/>
  </mappers>
</configuration>
```
## *[例] 快速入门*
```java
InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
// SqlSession sqlSession = sqlSessionFactory.openSession();  // 默认不自动提交事务
SqlSession sqlSession = sqlSessionFactory.openSession(true);  // autoCommit自动提交事务

// 代理模式，帮我们返回接口实现类对象
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
int n = mapper.insertUser();

// sqlSession.commit();  // 手动提交事务
```
# 查询数据

## 传入参数

MyBatis获取参数的主要方式：`${}`和`#{}`(推荐)

- `${}`的本质是字符串拼接，若为字符串类型或日期类型的字段，须手动加单引号；

- `#{}`的本质是占位符赋值，若为字符串类型或日期类型的字段，可自动加单引号；
```java
public interface UserMapper {
    User getUserByName(String usr);
    User checkLogin(String usr, String pwd);
    User checkLoginByMap(Map<String, Object> map);
    int insertUser(User user);
    User checkLoginByParam(@Param("usr") String usr, @Param("pwd") String pwd);
}
```
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">

  <!--单个参数，命名不限-->
  <select id="getUserByName" resultType="User">
    select * from t_user where username = #{usr}
    select * from t_user where username = '${usr}'
  </select>

  <!--多个参数，会将参数放在map中，arg0、arg1、param1、param2等为键，参数为值-->
  <select id="checkLogin" resultType="User">
    select * from t_user where username = #{arg0} and password = #{arg1}
    select * from t_user where username = '${param1}' and password ='${param2}'
  </select>

  <!--自定义map传参，通过键访问值即可 {usr: Jack, pwd: 123}-->
  <select id="checkLoginByMap" resultType="User">
    select * from t_user where username = #{usr} and password = #{pwd}
    select * from t_user where username = '${usr}' and password ='${pwd}'
  </select>

  <!--实体类传参，以属性的方式访问属性值即可-->
  <insert id="insertUser">
    insert t_user values (#{name}, #{age})
    insert t_user values ('${name}', ${age})
  </insert>

  <!--通过注解传参，会将参数放在map中，注解的值为键，参数为值-->
  <select id="checkLoginByParam" resultType="User">
    select * from t_user where username = #{usr} and password = #{pwd}
    select * from t_user where username = '${usr}' and password ='${pwd}'
  </select>

</mapper>
```
### 特殊参数

模糊查询
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">

  <!--不能用#{}，使用${}-->
  <select id="getUserByLike" resultType="user">
    select * from t_user where username like '%${usr}%'
  </select>

  <!--可用#{}-->
  <select id="getUserByLike" resultType="user">
    select * from t_user where username like concat('%', #{usr}, '%')
  </select>

  <!--常用-->
  <select id="getUserByLike" resultType="user">
    select * from t_user where username like "%"#{usr}"%"
  </select>

</mapper>
```
动态设置表名
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">

  <!--不能用#{}，使用${}-->
  <select id="getUserByTable" resultType="user">
    select * from ${table}
  </select>

</mapper>
```
## 获取结果
```java
public interface UserMapper {

    // 获取一个结果
    User getUserById(int id);

    // 获取多个结果
    List<User> getAllUser();

    // 用map接收结果，键为字段名，值为值
    Map<String, String> getUserByIdToMap(int id);

    // 获取多个map结果
    List<Map<String, String>> getAllUserToMapList();

    // 用map接收多个结果，并指定键
    @MapKey("id")  // 用id作为键
	Map<String, Object> getAllUserToMap();
}
```
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">
  <select id="getAllUserToMapList" resultType="map">
    select * from t_user
  </select>
</mapper>
```
### 获取自增主键
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">
  <!--useGeneratedKeys：使用自增主键 keyProperty：主键值赋值给id属性-->
  <insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
    insert into t_user values (null, 'admin')
  </insert>
</mapper>
```
### 字段名≠属性名

法一：取别名
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">
  <select id="getAllUser" resultType="user">
    select id, user_name userName from t_user
  </select>
</mapper>
```
法二：全局配置
```xml
<configuration>

 <!--全局配置-->
  <settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>  <!--下划线映射驼峰-->
  </settings>

</configuration>
```
法三：设置`resultMap`
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">

	<resultMap id="userMap" type="User">
    <id property="id" column="id"/>
    <result property="userName" column="user_name"/>
  </resultMap>

  <select id="getAllUser" resultMap="userMap">
    select * from t_user
  </select>
</mapper>
```
## 动态SQL

根据特定条件动态拼装SQL，解决拼接SQL语句字符串的痛点

### if
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">
  <select id="getUser" resultType="user">

    select * from t_user where 1=1
    <if test="name != null and name != ''">
      and name = #{name}
    </if>
    <if test="age != null and age != ''">
      and age = #{age}
    </if>

  </select>
</mapper>
```
### where

有内容时会生成where关键字，并且将内容前（内容后不行）多余的and、or去掉，没有内容时不会生成where
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">
  <select id="getUser" resultType="user">

    select * from t_user
    <where>
      <if test="name != null and name != ''">
        name = #{name}
      </if>
      <if test="age != null and age != ''">
        and age = #{age}
      </if>
    </where>

  </select>
</mapper>
```
### trim

若标签中没内容，标签没有效果，有内容时：

- `prefix`/`suffix`：内容前/后添加指定内容

- `prefixOverrides`/`suffixOverrides`：内容前/后去掉指定内容
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">
  <select id="getUser" resultType="user">

    select * from t_user
    <trim prefix="where" prefixOverrides="and|or">
      <if test="name != null and name != ''">
        name = #{name}
      </if>
      <if test="age != null and age != ''">
        and age = #{age}
      </if>
    </trim>

  </select>
</mapper>
```
### choose 🆚 when 🆚 otherwise

相当于switch...case...default
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">
  <select id="getUser" resultType="user">

    select * from t_user
    <where>
      <choose>
        <when test="name != null and name != ''">
        	name = #{name}
        </when>
        <when test="age != null and age != ''">
        	age = #{age}
        </when>
        <otherwise>
          id = 1
        </otherwise>
      </choose>
    </where>

  </select>
</mapper>
```
### foreach

批量删除
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">
  <delete id="deleteMany">

    <!--法一-->
    delete from t_user where id in
    <foreach collection="ids" item="id" separator="," open="(" close=")">
      #{id}
    </foreach>

    <!--法二-->
    delete from t_user where
    <foreach collection="ids" item="id" separator="or">
      id = #{id}
    </foreach>

  </delete>
</mapper>
```
批量添加
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">
  <insert id="insertMany">

    <!--法一-->
    insert into t_user values
    <foreach collection="users" item="user" separator=",">
      (null, #{user.name}, #{user.age})
    </foreach>

    <!--法二-->
    delete from t_user where
    <foreach collection="ids" item="id" separator="or">
      id = #{id}
    </foreach>

  </insert>
</mapper>
```
### sql
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">

  <sql id="userColumns">id,name,age</sql>

  <select id="getUser" resultType="user">
    select <include refid="userColumns"/> from t_user
  </select>

</mapper>
```
# 关系映射

多对一：员工🆚部门

一对多：部门🆚员工

## 多对一
```xml
<mapper namespace="com.xxx.mybatis.mappers.EmpMapper">

	<resultMap id="empAndDeptMap" type="Emp">
    <id property="eid" column="eid"/>
    <result property="empName" column="emp_name"/>

    <!--法一：级联属性赋值-->
    <result property="dept.did" column="did"/>
    <result property="dept.deptName" column="dept_name"/>

    <!--法二：association标签-->
    <association property="dept" javaType="Dept">
      <id property="did" column="did"/>
    	<result property="deptName" column="dept_name"/>
    </association>

  </resultMap>

  <select id="getEmpAndDeptById" resultMap="empAndDeptMap">
    select * from t_emp left join t_dept on t_emp.did = t_dept.did where t_emp.eid = #{eid}
  </select>
</mapper>
```
法三：分步查询（常用）
```xml
<mapper namespace="com.xxx.mybatis.mappers.EmpMapper">

	<resultMap id="empAndDeptMap" type="Emp">
    <id property="eid" column="eid"/>
    <result property="empName" column="emp_name"/>
    <association property="dept" select="com.xxx.mybatis.mappers.DeptMapper.getEmpAndDeptById2" column="did"/>
  </resultMap>

  <select id="getEmpAndDeptById1" resultMap="empAndDeptMap">
    select * from t_emp where eid = #{eid}
  </select>
</mapper>
```
```xml
<mapper namespace="com.xxx.mybatis.mappers.DeptMapper">
  <select id="getEmpAndDeptById2" resultType="Dept">
    select * from t_dept where did = #{did}
  </select>
</mapper>
```
### 懒加载

分步查询优点：可以实现延迟加载，须在主配置文件中设置：

- lazyLoadingEnabled：延迟加载全局开关，开启后所有对象延迟加载；

- aggressiveLazyLoading：开启时任何方法的调用都会加载对象的所有属性，否则按需加载

开启了全局延迟加载后，还可以在映射文件中手动设置按需加载，通过association、collection标签中的`fetchType=lazy(延迟加载)/eager(立即加载)`属性设置当前的分步查询是否延迟加载
```xml
<configuration>
  <settings>
    <setting name="lazyLoadingEnabled" value="true"/>
  </settings>
</configuration>
```
## 一对多
```xml
<mapper namespace="com.xxx.mybatis.mappers.DeptMapper">

	<resultMap id="deptAndEmpMap" type="Dept">
    <id property="did" column="did"/>
    <result property="deptName" column="dept_name"/>

    <!--法一：collection标签-->
    <collection property="emps" ofType="Emp">
      <id property="eid" column="eid"/>
    	<result property="empName" column="emp_name"/>
    </collection>
  </resultMap>

  <select id="getDeptAndEmpById" resultMap="deptAndEmpMap">
    select * from t_dept left join t_emp on t_emp.did = t_dept.did where t_emp.did = #{did}
  </select>
</mapper>
```
法二：分步查询
```xml
<mapper namespace="com.xxx.mybatis.mappers.DeptMapper">

	<resultMap id="deptAndEmpMap" type="Dept">
    <id property="did" column="did"/>
    <result property="deptName" column="dept_name"/>
    <collection property="emps" select="com.xxx.mybatis.mappers.EmpMapper.getDeptAndEmpById2" column="did"/>
  </resultMap>

  <select id="getDeptAndEmpById1" resultMap="deptAndEmpMap">
    select * from t_dept where did = #{did}
  </select>
</mapper>
```
```xml
<mapper namespace="com.xxx.mybatis.mappers.EmpMapper">
  <select id="getDeptAndEmpById2" resultType="Emp">
    select * from t_emp where did = #{did}
  </select>
</mapper>
```
# 缓存

## 一级缓存

一级缓存是SqlSession级别，默认开启。通过同一个SqlSession查询的数据会被缓存，下次查询相同的数据会从缓存中直接取，而不重新访问数据库

一级缓存失效的情况：

- 不同的SqlSession对应不同的一级缓存；

- 同一个SqlSession但是查询条件不同；

- 同一个SqlSession两次查询期间执行了任意增删改操作；

- 同一个SqlSession两次查询期间手动清空了缓存`sqlSession.clearCache()`；

## 二级缓存

二级缓存是SqlSessionFactory级别，需手动开启。通过同一个SqlSessionFactory创建的SqlSession查询的结果会被缓存；此后若再次执行相同的查询语句，结果就会从缓存中取

二级缓存失效的情况：

- 两次查询之间执行了任意增删改操作，会使一级、二级缓存同时失效

二级缓存开启的条件（缺一不可）：

- 在主配置文件中，设置`cacheEnabled="true"`，默认true；

- 在映射文件中设置`cache`标签（`eviction`(回收策略)/`flushInterval`(刷新间隔ms)/`size`(引用数目)/`readOnly`(只读)）；

- 二级缓存必须在SqlSession关闭`sqlSession.close()`或提交`sqlSession.commit()`之后有效；

- 查询的数据所转换的实体类必须实现序列化接口；
```xml
<mapper namespace="com.xxx.mybatis.mappers.UserMapper">

  <cache />

  <select id="getUser" resultType="user">
    select * from t_user where id = #{id}
  </select>
</mapper>
```
## 查询顺序

- 先查询二级缓存，因为二级缓存中可能会有其它程序已经查出来的数据；

- 如果二级缓存没有命中，查询一级缓存；

- 如果一级缓存没有命中，查询数据库；

- SqlSession关闭后，一级缓存中的数据会写入二级缓存；
