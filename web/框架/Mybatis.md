# MyBatis

## Mybatis基础

### Mybatis和Hibernate的区别

Mybatis和hibernate不同，它不完全是一个ORM框架，因为MyBatis需要程序员自己编写Sql语句。mybatis可以通过XML或注解方式灵活配置要运行的sql语句，并将java对象和sql语句映射生成最终执行的sql，最后将sql执行的结果再映射生成java对象。

Mybatis学习门槛低，简单易学，程序员直接编写原生态sql，可严格控制sql执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，例如互联网软件、企业运营类软件等，因为这类软件需求变化频繁，一但需求变化要求成果输出迅速。但是灵活的前提是mybatis无法做到数据库无关性，如果需要实现支持多种数据库的软件则需要自定义多套sql映射文件，工作量大。

Hibernate对象/关系映射能力强，数据库无关性好，对于关系模型要求高的软件（例如需求固定的定制化软件）如果用hibernate开发可以节省很多代码，提高效率。但是Hibernate的学习门槛高，要精通门槛更高，而且怎么设计O/R映射，在性能和对象模型之间如何权衡，以及怎样用好Hibernate需要具有很强的经验和能力才行。

对于性能要求不太苛刻的系统，比如管理系统，ERP等推荐使用Hibernate；对于性能要求高，响应快，灵活的系统则推荐使用Mybatis。

### Mybatis的核心组件

SqlSessionFactoryBuilder（构造器）：它会根据配置或者代码来生成SqlSessionFactory，采用的是分布构建的Builder模式

SqlSessionFactory（工厂接口）：依赖它来生成SqlSession，使用的是工厂模式

SqlSession（会话）：即可以发送Sql执行返回结果，也可以获取Mapper的接口。在现有的技术中，一般我们会让其在业务逻辑代码中“消失”，而使用的是MyBatis提供的SQL Mapper接口编程技术，他能提高代码的可读性和可维护性。

SQL Mapper（映射器）：MyBatis新设计存在的组件，它由一个Java接口和XML文件（或注解）构成，需要给出对应的SQL和映射规则。它负责发送SQL去执行，并返回结果。

### Mybatis编程步骤

+ 创建 SqlSessionFactory 对象。
+ 通过 SqlSessionFactory 获取 SqlSession 对象。
+ 通过 SqlSession 获得 Mapper 代理对象。
+ 通过 Mapper 代理对象，执行数据库操作。
+ 执行成功，则使用 SqlSession 提交事务。
+ 执行失败，则使用 SqlSession 回滚事务。
+ 最终，关闭会话。

#### SqlSessionFactory

使用MyBatis首先是使用配置或者代码去生产SqlSessionFactory，而MyBatis提供了构造器SqlSessionFactoryBuilder。采用Builder模式，它提供了一个类org.apache.ibatis.session.Configuration作为引导，采用Builder模式。Factory的具体初始化配置在Configuration类里面完成的。可通过读取XML或者代码的形式来配置SqlSessionFactory，当配置了XML或者提供代码后，MyBatis会读取配置文件，通过Configuration来构建整个MyBatis上下文。关系图如下：

![](F:\mycode\knowledgeArrangement\web\sqlsessionfactorybuilder.png)

SqlSessionManager使用在多线程环境中，它的具体实现依靠DefaultSqlSessionFactory。SqlSessionFactory唯一的作用是生产SqlSession，往往用单例来处理。

##### 使用XML构建SqlSessionFactory

```xml
<configuration>
    <typeAliases>
        <typeAlias alias="role" type="com.xxx.xxx.xxx.Role" />
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/ssm" />
                <property name="username" value="root" />
                <property name="password" value="123456" />
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/xxx/xxx/xx/mapper/RoleMapper.xml" />
    </mappers>
</configuration>
```

<typaAlias>元素定义了一个别名，这样定义后，在MyBatis上下文中就可以使用别名来替代全限定名了

<environment>元素的定义描述的是数据库，<transactionManager>元素是配置事务管理器。

<mapper>元素代表引入哪些映射器

#### SqlSession

在MyBatis中有两个实现类，DefaultSqlSession和SqlSessionManager。DefaultSqlSession是单线程使用的，SqlSessionManager在多线程环境下使用。SqlSession的作用类似于一个JDBC中的Connection对象，代表着一个连接资源的启用。它的作用有三个：

1. 获取Mapper接口
2. 发送SQL给数据库
3. 控制数据库事务

```Java
SqlSession sqlSession = null;
try {
    sqlSession = SqlSessionFactory.openSession();
    // some code...
    // 提交事务
    sqlSession.commit();
} catch(Exception ex) {
    // 异常，回滚事务
    sqlSession.rollback();
} finally {
    // 确保资源被顺利关闭
    if (sqlSession != null) 
        sqlSession.close();
}
```



#### 映射器

映射器由一个接口和对应的XML文件（或者注解）组成，它可以配置一下内容：

- 描述映射规则
- 提供SQL语句
- 配置缓存
- 提供动态SQL

```Java
// 定义一个POJO
package com.learn.ssm.chapter3.pojo;
public class Role {
    private Long id;
    private String roleName;
    private String note;
    /**setter and getter**/
}
```

##### 用XML实现映射器

先定义一个映射器接口

```Java
package com.learn.ssm.chapter3.mapper;
public interface RoleMapper {
    public Role getRole(Long id);
}
```

在用XML方式创建SqlSession的配置文件中有这样一段代码：

```xml
<mapper resource="com/learn/ssm/chapter3/mapper/RoleMapper.xml" />
```

它的作用是引入一个XML文件。用XML的方式创建映射器，代码如下：

```xml
<mapper namespace="com.learn.ssm.chapter3.mapper.RoleMapper">
    <select id="getRole" parameterType="long" resultType="role">
        select id, role_name as roleName, note from t_role where id = #{id}
    </select>
</mapper>
```

##### 注解实现映射器

```Java
import com.learn.ssm.chapter3.pojo.Role;
public interface RoleMapper2 {
    @Select("select id, role_name as roleName, note from t_role where id=#{id}")
    public getRole(Long id);
}
```

如果它和XML方式同时定义时,XML方式将覆盖掉注解方式,MyBatis官方推荐使用的是XML方式

##### SqlSession发送SQL

```Java
Role role = (Role)sqlSession.selectOne("com.learn.ssm.chapter3.mapper.RoleMapper.getRole", 1L);
```

selectOne方法表示用查询并且返回一个对象.而参数是一个Strting对象和Object对象

##### 用Mapper接口发送SQL

```Java
RoleMapper roleMapper = sqlSession.getMapper(RoleMapper.class);
Role role = roleMapper.getRole(1L);
```

推荐采用SqlSession获取Mapper的方式,理由如下:

- 使用Mapper接口编程可以消除SqlSession带来的功能性代码,提高可读性,而SqlSession发送SQL,需要一个SQL id去匹配SQL,比较晦涩难懂.使用Mapper接口,则完全是面向对象的语言,更能体现业务的逻辑.
- 使用Mapper的方式,更加不容易出错

### 各组件的生命周期

SqlSessionFactoryBuilder的作用在于创建SqlSessionFactory,创建成功后builder就失去了作用,它只能存在于创建SqlSessionFactory的方法中,而不要让其长期存在.SqlSessionFactory是一个对数据库的连接池,可以认为SqlSessionFactory的生命周期就等同于MyBatis的应用周期.SqlSessionFactory是一个单例,在应用中被共享.SqlSession相当于一个数据库连接对象,你可以在一个事务里面执行多条SQL,然后提交或者回滚事务.所以SqlSession应该存活在一个业务请求中,处理完整个请求后,应该关闭这条链接,避免占用资源.Mapper是一个接口,它的最大生命周期至多和SqlSession保持一致,Mapper代表的是一个请求中的业务处理,一旦完成了相关业务,就应该废弃它.

mybatis中#{}与${}的区别

`#{}`：用于变量的传递,`#{}`是 SQL 的参数占位符，Mybatis 会将 SQL 中的 `#{}` 替换为 `?` 号.在 SQL 执行前会使用 PreparedStatement 的参数设置方法，按序给 SQL 的 `?`号占位符设置参数值

+ 使用预编译的方式将参数设置到sql语句中
+ 使用的原生jdbc中的prepareStatement
+ 取出的值以字符串的形式拼接（会加''）
+ 能够在一定程度上防止sql注入的风险（无法避免%的问题）

`${}`：用于简单的字符串拼接

+ 不适用预编译模式
+ 取出的值**直接**拼装在sql语句中
+ 会有sql注入问题
+ `${}`一般用于传入数据库的对象，例如表名，因为#以字符串形式拼接

### 如果实体类中的属性名和表中的字段名不一样，怎么处理

+ 第一种， 通过在查询的 SQL 语句中定义字段名的别名，让字段名的别名和实体类的属性名一致

+ 第二种，是第一种的特殊情况。大多数场景下，数据库字段名和实体类中的属性名差，主要是前者为下划线风格，后者为驼峰风格。可以通过Mybatis的配置解决

  ```xml
  <settings>
      <setting name="mapUnderscoreToCamelCase" value="true" />
  </settings>
  ```

+ 通过 `<resultMap>` 来映射字段名和实体类属性名的一一对应的关系

  ```xml
  <resultMap id="BaseResultMap" type="com.pinyougou.pojo.TbAddress" >
    <id column="id" property="id" jdbcType="BIGINT" />
    <result column="user_id" property="userId" jdbcType="VARCHAR" />
  </resultMap>
  ```

+ 定义别名

### MyBatis配置

所有元素代码清单如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration><!--配置-->
    <properties /><!--属性-->
    <settings /><!--设置-->
    <typeAliases /><!--类型命名-->
    <typeHandlers /><!--类型处理器-->
    <objectFactory /><!--对象工厂-->
    <pluginss /><!--插件-->
    <environments>
        <environment>
            <transactionManager /><!--事务管理器-->
            <datatSource /><!--数据源-->
        </environment>
    </environments>
    <databaseIdProvider /><!--数据库厂商标识-->
    <mappers /><!--映射器-->
</configuration>
```

需要注意的是,MyBatis配置项的顺寻不能颠倒.如果颠倒了它们的顺序,那么在MyBatis启动阶段就会发生异常,导致程序无法运行.

### Mybatis的映射文件的常见标签

顶级标签：

+ `cache` – 给定命名空间的缓存配置。
+ `cache-ref` – 其他命名空间缓存配置的引用。
+ `resultMap` – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
+ `sql` – 可被其他语句引用的可重用语句块。
+ `insert` – 映射插入语句
+ `update` – 映射更新语句
+ `delete` – 映射删除语句
+ `select` – 映射查询语句

动态SQL标签:动态 SQL ，可以让我们在 XML 映射文件内，以 XML 标签的形式编写动态 SQL ，完成逻辑判断和动态拼接 SQL 的功能。mybatis是使用 OGNL 的表达式，从 SQL 参数对象中计算表达式的值，根据表达式的值动态拼接 SQL ，以此来完成动态 SQL 的功能

+ `<if />` 判断的条件是否满足

  ```xml
  <select id="findActiveBlogWithTitleLike"
   resultType="Blog">
      SELECT * FROM BLOG 
      WHERE state = ‘ACTIVE’ 
      <if test="title != null">
          AND title like #{title}
      </if>
  </select>
  ```

+ `<choose />`、`<when />`、`<otherwise />`这三个一起使用，类似于switch语句

  ```xml
  <select id="findActiveBlogLike"
       resultType="Blog">
    SELECT * FROM BLOG WHERE state = ‘ACTIVE’
    <choose>
      <when test="title != null">
        AND title like #{title}
      </when>
      <when test="author != null and author.name != null">
        AND author_name like #{author.name}
      </when>
      <otherwise>
        AND featured = 1
      </otherwise>
    </choose>
  </select>
  ```

+ `<trim />、<where />、<set />`:这三个是为了对if语句进行补充。where保证包括的部分至少有一个if满足时才返回子句。trim用于自定义where，set用于更新语句，会动态前置 SET 关键字，同时也会删掉无关的逗号。trim也可以自定义set

  ```xml
  <select id="findActiveBlogLike" resultType="Blog">
      SELECT * FROM BLOG
      WHERE
      <if test="state != null">
        state = #{state}
      </if>
      <if test="title != null">
        AND title like #{title}
      </if>
      <if test="author != null and author.name != null">
        AND author_name like #{author.name}
      </if>
  </select>
  ```

  上面的主语句中如果一个条件都没有匹配，会出现`SELECT * FROM BLOG WHERE`这种情况，如果匹配了部分则出现`SELECT * FROM BLOG WHERE AND title like ‘someTitle’`这种AND前面没有语句。这种情况就需要where。where包括的语句在至少有一个条件满足时才返回子句，并且会去除多余的AND或OR

  ```xml
  <select id="findActiveBlogLike"
       resultType="Blog">
    SELECT * FROM BLOG
    <!-- where包括的部分在至少有一个子元素的条件返回 SQL 子句的情况下才去插入“WHERE”子句 -->
    <where>
      <if test="state != null">
           state = #{state}
      </if> 
      <if test="title != null">
          AND title like #{title}
      </if>
      <if test="author != null and author.name != null">
          AND author_name like #{author.name}
      </if>
    </where>
  </select>
  ```

  如果where没有满足需求，可以用**trim来自定义where**。

  ```xml
  <trim prefix="WHERE" prefixOverrides="AND |OR ">
    ...
  </trim>
  ```

  set有类似于where的作用，但用于update语句，set 元素可以用于动态包含需要更新的列

  ```xml
  <update id="updateAuthorIfNecessary">
    update Author
      <set>
        <if test="username != null">username=#{username},</if>
        <if test="password != null">password=#{password},</if>
        <if test="email != null">email=#{email},</if>
        <if test="bio != null">bio=#{bio}</if>
      </set>
    where id=#{id}
  </update>
  ```

  set 元素会动态前置 SET 关键字，同时也会删掉无关的逗号，因为用了条件语句之后很可能就会在生成的 SQL 语句的后面留下这些逗号

+ `<foreach />`对一个集合进行遍历

  ```xml
  <select id="selectPostIn" resultType="domain.blog.Post">
    SELECT *
    FROM POST P
    WHERE ID in
    <foreach item="item" index="index" collection="list"
        open="(" separator="," close=")">
          #{item}
    </foreach>
  </select>
  ```

  允许指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头与结尾的字符串以及在迭代结果之间放置分隔符

+ `<bind />`可以从 OGNL 表达式中创建一个变量并将其绑定到上下文

```xml
<select id="selectBlogsLike" resultType="Blog">
    <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
    SELECT * FROM BLOG
    WHERE title LIKE #{pattern}
</select>
```

### 传递多个参数的方式

- 使用map接口传递参数

把接口方法定义为:

```Java
public List<Role> findRolesByMap(Map<String, Object> parameterMap);
```

此时传递给映射器的是一个map对象,使用它在SQL中设置对应的参数,代码如下:

```xml
<select id="findRolesByMap" parameterType="map" resultType="role">
    select id, role_name as roleName, note from t_role where role_name like concat('%', #{roleName}, '%') and note like concat('%', #{note}, '%')
</select>
```

要求参数roleName和note都是map的键.但是用的不多,主要原因如下:

1. map是一个键值对应的集合,使用者要通过阅读它的键,才能明了其作用
2. map不能限定其传递的数据类型,因此业务性质不强,可读性差

- 使用注解传递多个参数

通过注解@param(org.apache.ibatis.annotation.Param),把接口方法定义为:

```Java
public List<Role> findRolesByAnnotation(@Param("roleName") String rolename, @Paran("note") String note);
```

此时映射文件的代码为:

```xml
<select id="findRolesByAnnotation" resultType="role">
    select id, role_name as roleName, note from t_role where role_name like concat('%', #{roleName}, '%') and note like concat('%', #{note}, '%')
</select>
```

此时无需指定parameterType属性

- 通过Java Bean传递多个参数

先定义一个参数的POJO

```Java
public class RoleParams {
    private String roleName;
    private String note;
    /** setter and getter **/
}
```

此时把接口方法定义为:

```Java
public List<Role> findRolesByBean(RoleParams, roleParam);
```

此时,映射文件改为:

```xml
<select id="findRolesByBean" parameterType="com.learn.ssm.xxx.RoleParams" resultType="role">
    select id, role_name as roleName, note from t_role where role_name like concat('%', #{roleName}, '%') and note like concat('%', #{note}, '%')
</select>
```

- 混合使用

```java
public class PageParams {
    private int start;
    private int limit;
    /**setter and getter**/
}

// 此时接口设计如下
public List<Role> findByMix(@Param("params") RoleParams roleParams, @Param("page") PageParam pageParam)
```

这样映射文件修改为:

```xml
<select id="findByMix" resultType="role">
    select id, role_name as roleName, note from t_role where role_name like concat('%', #{params.roleName}, '%')
     and note like concat('%', #{params.note}, '%') limit #{page.start}, #{page.limit}
</select>
```

### 使用resultMap映射结果集

自动映射和驼峰映射规则比较简单,无法定义多的属性,比如typeHandler、级联等。为了支持复杂的映射，select元素提供了resultMap属性，如下所示：

```xml
<mapper namespace="com.learn.ssm.chapter5.mapper.RoleMapper">
    <resultMap id="roleMap" type="role">
        <id property="id" column="id" />
        <result property="roleName" column="role_name" />
        <result property="note" column="note" />
    </resultMap>
    <select id="getRoleUseResultMap" parameterType="long" resultMap="roleMap" />
        select id, role_name, note from t_role where id = #{id}
    </select>
</mapper>
```

resultMap元素定义了一个roleMap，它的属性id代表它的标识，type代表哪个类作为其映射的类，可以是别名或者全限定名；它的子元素id代表resultMap的主键，而result代表其属性，id和result元素的属性property代表POJO的属性名称，而column代表SQL的列名。把POJO的属性和SQL的列名做对应；select元素中的属性resultMap制定了采用哪个resultMap作为其映射规则。

### 分页参数

MyBatis内置了一个专门处理分页的类——RowBounds。其初始化器如下：

```java
public RowBounds(int offset, int limit) {
    this.offset = offset;
    this.limit = limit;
}
```

offset属性是偏移量，即从第几行开始对取记录。limit是限制条数，从源码可知，默认值为0和最大整数。使用它非常简单，只要给接口增加一个RowBounds参数即可。

```Java
public List<Role> findByRowBounds(@Param("roleName") String rolename, @Param("note") String note, RowBounds rowBounds);
```

对于SQL而言，完全不需要有与RowBounds相关的信息，它是MyBatis的一个附加参数，MyBatis会自动识别它，据此进行分页。

```xml
<select id="findByRowBounds" resultType="role">
    select id, role_name as roleName, note from t_role where role_name like concat('%', #{roleName}, '%') and note like concat('%', #{note} '%')
</select>
```

### insert元素——插入语句

```xml
<insert id="insertRole" parameterType="role" >
	insert into t_role(role_name, note) values(#{roleName}, #{note})
</insert>
```

id标识出这条SQL，结合命名空间让MyBatis能找到它

parameterType代表传入参数类型

##### 主键回填

JDBC中的Statement对象在执行插入的SQL后，通过getGeneratedKey方法获得数据库生成的主键（需要数据库驱动的支持）。在insert语句中有一个开关属性useGeneratedKeys，用来控制是否打开这个功能。还要配置属性keyProperty和keyColumn，告诉系统把生成的主键放入哪个属性中，如果存在多个主键，就要用，将它们隔开

```xml
<insert id="insertRole" parameterType="role" useGeneratedKeys="true" keyProperty="id">
    insert into t_role(role_name, note) values(#{roleName}, #{note})
</insert>
```

useGeneratedKeys代表用JDBC的statement对象的getGeneratedKeys方法返回主键，而keyProperty则代表用哪个POJO的属性去匹配这个主键

##### 自定义主键

```xml
<insert id="insertRole" parameterType="role" >
    <selectKey keyProperty="id" resultType="long" order="BEFORE" >
        select if (max(id) == null, 1, max(id)+3) from t_role 
    </selectKey>
    insert into t_role(id, role_name, note) values(#{id}, #{roleName}, #{note})
</insert>
```

keyProperty指定了采用哪个属性作为POJO的主键。resultType告诉MyBatis返回一个long类型的结果集，而order设置为BEFORE说明它将于当前定义的SQL前执行。

### Update元素和Delete元素

和insert属性差不多，执行完也会返回一个整数，用以表示该SQL语句影响了数据库的记录行数

```xml
<update id="updateRole" parameterType="role" >
    update t_role set role_name = #{roleName}, note = #{note} where id = #{id}
</update>
<delete id="deleteRole" parameterType="long">
    delete from t_role where id=#{id}
</delete>
```

### SQL元素

可以定义一条SQL的一部分，方便后面的SQL引用它。

```xml
// 定义
<sql id="roleCols">id, role_name, note</sql>

// SQL使用
<select id="getRole" parameterType="long" resultMap="roleMap">
    select <include refid="roleCols" /> from t_role where id = #{id}
</select>
<insert id="insertRole" parameterType="role">
    <selectKey keyProperty="id" resultType="long" order="BEFORE" statementType="PREPARED">
        select if (max(id) == null, 1, max(id)+3) from t_role
    </selectKey>
    insert into t_role(<include refid="roleCols" />) values(#{id}, #{roleName}, #{note})
</insert>

// sql元素还支持变量传递
<sql id="roleCols">
    ${alias}.id, ${alias}.role_name, ${alias}.note
</sql>

<select id="getRole" parameterType="long" resultMap="roleMap">
    select <include refid="roleCols"><property name="alias" value="r" /></include>
    from t_role r where id=#{id}
</select>
```

### resultMap元素

resultMap的子元素如下：

```xml
<resultMap>
    <constructor>
        <idArg />
        <arg />
    </constructor>
    <id />
    <result />
    <association />
    <collection />
    <discriminator>
        <case />
    </discriminator>
</resultMap>
```

<constructor>用于配置有参构造方法

```xml
<!-- public RoleBean(Integer id, String roleName) -->
<constructor>
    <idArg column="id" javaType="int" />
    <arg column="role_name" javaType="string" />
</constructor>
```

这样MyBatis就会使用对应的构造方法来构造POJO

id元素表示哪个列是主键，允许多个主键，多个主键则为联合主键。result是配置POJO到SQL列名的映射关系。

##### 使用POJO存储结果集

可以使用自动映射，正如使用resultType属性一样，但是有时候需要更为复杂的映射或者级联，这时使用select语句的resultMap属性配置映射集合。只是使用前要配置类似的resultMap

```xml
<resultMap id="roleResultMap" type="com.learn.chapter4.pojo.Role">
    <id property="id" column="id" />
    <result property="roleName" column="role_name" />
    <result property="note" column="note" />
</resultMap>
```

这样就可以使用resultMap了：

```xml
<select parameterType="long" id="getRole" resultMap="roleResultMap" >
    select id, role_name, note from t_role where id = #{id}
</select>
```

由此可见，SQL语句的列名和roleResultMap的column是一一对应的

### 级联

级联不是必须的，级联的好处是获取关键数据十分便捷，但是级联过多会增加系统的复杂度，同时降低系统的性能，所以当级联的层级超过3层时，就不要考虑使用级联了，因为这样会导致多个对象的关联，导致系统的耦合、复杂和难以维护。

##### MyBatis中的级联

分为3种：

- 鉴别器：它是一个根据某些条件决定采用具体实现类级联的方案。
- 一对一级联
- 一对多级联

MyBatis不支持多对多级联

使用association元素，可以实现1对1级联关系

```xml
<association property="task" column="task_id" select="com.xxx.xxx.mapper.TaskMapper.getTask" />
```

association元素代表着一对一级联的开始。property属性代表映射到POJO属性上。select配置是命名空间+SQL id的形式，这样便可以指向对应Mapper的SQL，MyBatis会通过对应的SQL将数据查询回来。column代表SQL的列，用作参数传递给select属性制定的SQL，如果是多个参数，就用逗号隔开。

使用collection元素可以实现一对多级联

```xml
<collection property="workCard" column="id" select="com.xxx.xxx.mapper.WorkCardMapper.getWorkCardByEmpId" />
```

其select元素指向SQL，将通过column制定的SQL字段作为参数进行传递，然后将结果返回给POJO的列表属性。

```xml
<resultMap type="com.xxx.xxxx.pojo.Employee" id="employee">
    <id column="id" property="id" />
    <result column="real_name" property="realName" />
    <result column="sex" property="sex" typeHandler="com.xxx.xxxx.typeHandler.sexTypeHandler" />
    <result column="birthday" property="birthday" />
    <result column="mobile" property="mobile" />
    <result column="email" property="email" />
    <result column="position" property="position" />
    <result column="note" property="note" />
    <association property="workCard" column="id" select="com.xxx.xxxx.mapper.WorkCardMapper.getWorkCardByEmpId" />
    <collection property="employeeTaskList" column="id" select="com.xxx.xxxx.mapper.EmployeeTaskMapper.getEmployeeTaskByEmpId" />
	<discriminator javaType="long" column="sex">
    	<case value="1" resultMap="maleHealthFormMapper" />
    	<case value="2" resultMap="femaleHealthFormMapper" />
	</discriminator>
</resultMap>
<resultMap type="com.xxx.xxxx.pojo.FemaleEmployee" id="femaleHealthFormMapper" extends="employee">
    <association property="femaleHealthForm" column="id" select="com.xxx.xxxx.mapper.FemaleHealthFormMapper.getFemaleHealthForm" />
</resultMap>
<resultMap type="com.xxx.xxxx.pojo.MaleEmployee" id="maleHealthFormMapper" extends="employee">
    <association property="maleHealthForm" column="id" select="com.xxx.xxxx.mapper.FemaleHealthFormMapper.getMaleHealthForm" />
</resultMap>
```

discriminator元素，鉴别器，它的属性column代表使用哪个字段进行鉴别， 注意extends元素。

##### N+1问题

级联会导致性能问题，一些不常用的信息的加载会导致数据库资源的损耗和系统性能的下降。

假设现在有N个关联关系完成了级联，那么只要再加入一个关联关系，就变成了N+1个级联，所有的级联SQL都会被执行，显然就会有很多并不是我们关系的数据被取出，这样会造成很大的资源浪费，这就是N+1问题，尤其在那些需要高性能的互联网系统中，这往往是不被允许的。

为了应对N+1问题，MyBatis提供了延迟加载功能。

##### 延迟加载

一次性将常用的级联数据通过SQL直接查询出来，而对于那些不常用的级联数据不要取出，而是等待要用时才取出，这些不常用的级联数据可以采用了延迟加载的功能。

settings配置中存在两个元素可以配置级联。

lazyLoadingEnable: 延迟加载的全局开关，当开启时，所有关联对象都会被延迟加载，在特定关联关系中，可通过设置fetchType属性来覆盖该项的开关状态。默认为False。

aggressiveLazyLoading：当启用时，对任意延迟属性的调用会使带有延迟加载属性的对象完整加载；反之，每种属性按需加载。false

修改这两个配置项：

```xml
<settings>
    <setting name="lazyLoadingEnable" value="true" />
    <setting name="aggressiveLazyLoading" value="true" />
</settings>
```

aggressiveLazyLoading配置项是一个层级开关，当设置为true时，它是一个开启了层级开关的延迟加载，如果为false，那么MyBatis按需加载。

选项lazyLoadingEnable决定是否开启延迟加载，而选项aggressiveLazyLoading则控制是否采用层级加载。

如果想要单独设置，可使用fetchType属性。fetchType属性出现在级联元素（association和collection中，discriminator没有这个属性可以设置）

它有两个值：

- eager：获得当前POJO后立即加载对应的数据。
- lazy：获得当前POJO后延迟加载对应的数据。

```xml
<collection property="employTaskList" column="id" fetchType="eager" select="com.xxx.xxxx.mapper.EmployeeTaskMapper.getEmployeeTaskByEmpId" />
```

##### 另一种级联

另一种级联方式是建立在SQL表连接（left join）的基础之上的，

```xml
<collection property="employeeTaskList" ofType="com.xxx.xxxx.pojo.EmployeeTask" column="id"></collection>
```

ofType用来声明实体映射

这样可以完全消除N+1问题，但是也会引发其他问题：

1. SQL比较复杂；
2. 所需配置比之前复杂得多；
3. 一次性将所有数据取出会造成内存的浪费；
4. 日后维护带来困难

##### 多对多级联

现实中多对多级联相当复杂，更多的是拆分为两个一对多的关系

### 缓存

缓存一般都是放在可告诉读写的存储器上，比如服务器的内存，它能够有效提高系统的性能。但是内存和高速缓存处理器的空间有限，所以一般只会将那些常用的且命中率高的数据缓存起来，不缓存那些不常用且命中率低的数据缓存。 

一级缓存是在SqlSession上的缓存，二级缓存是在SQL SessionFactory上的缓存。默认是开启一级缓存，不需要POJO对象可序列化（实现java.io.Serializable接口）。

当一个SqlSession第一次通过SQL和参数获取对象后，它就会将其缓存起来，如果下次的SQL和参数都没有发生变化，并且缓存没有超时或者声明需要刷新时，那么它就会从缓存中获取数据，而不是通过SQL获取

对于不同的SqlSession对象是不能共享的。为了使SqlSession对象之间共享相同的缓存，有时就要开启二级缓存，开启二级缓存只需要在映射文件上加入代码：

```xml
<cache />
```

这是MyBatis会序列化和反序列化对应的POJO，也就是要求POJO是一个可序列化的对象，那么它就必须实现Serialization接口。

##### 缓存配置项、自定义和引用

可使用自定义缓存，实现类需要实现Cache接口：

```java
package org.apache.ibatis.cache;

public interface Cache {
    // 获取缓存id
    String getId();
    // 保存对象key为键，value为值
    void putObject(Object key, Object value);
    // 获取缓存对象
   	Object getObject(Object key);
    // 删除缓存对象
    Object removeObject(Object key);
    // 清除缓存
    void clear();
    // 获取缓存大小
    int getSize();
    // 获取读写锁，需要考虑多线程的场景
    ReadWriteLock getReadWriteLock();
}
```

在现实中，我们可以使用Redis、MongoDB或者其他常用的缓存。

可以如下配置

```xml
<cache type="com.xxx.xxxx.cache.RedisCache">
    <property name="host" value="localhost" />
</cache>
```

如果在其他映射器中需要使用同样的配置，则可以引用缓存的配置：

```xml
<cache-ref namespace="com.xxx.xxxx.mapper.RoleMapper" />
```

### 动态SQL

##### if元素

```xml
<select id="findRoles" parameterType="string" resultMap="roleResultMap">
    select role_no, role_name, note from t_role where 1=1
    <if test="roleName != null and roleName != ''">
        and role_name like concate('%', #{roleName}, '%')
    </if>
</select>
```

where 1=1 用于程序根据用户选择的不同拼凑where条件时用的

##### choose、when、otherwise元素

```xml
<select id="findRoles" parameterType="role" resultMap="roleResultMap">
    select role_no, role_name, note from t_role where 1=1
    <choose>
        <when test="roleNo != null and roleNo != ''">
            AND role_no = #{roleNo}
        </when>
        <when test="roleName != null and roleName != ''">
            AND role_name like concat('%', #{roleName}, '%')
        </when>
        <otherwise>
            AND note is not null
        </otherwise>
    </choose>
</select>
```

##### trim、where、set元素

```xml
<select id="findRoles" parameterType="role" resultMap="roleResultMap">
	select role_no, role_name, note from t_role
    <where>
        <if test="roleName != null and roleName != ''">
            and role_name like concate('%', #{roleName}, '%')
        </if>
        <if test="roleName != null and roleName != ''">
            and note like concat('%', #{note}, '%')
        </if>
    </where>
</select>

<!-- trim元素表明要去掉一些特殊的字符串，prefix代表的是语句的前缀，而prefixOverrides代表的是需要去掉哪种字符串 -->
<select id="findRoles" parameterType="role" resultMap="roleResultMap">
	select role_no, role_name, note from t_role 
    <trim prefix="where" prefixOverrides="and">
        <if test="roleName != null and roleName != ''">
            and role_name like concate('%', #{note}, '%')
        </if>
    </trim>
</select>

<update id="updateRole" parameterType="role">
	update t_role
    <set>
        <if test="roleName != null and roleName != ''">
            role_name = #{roleName},
        </if>
        <if test="note != null and note !=''">
        	note = #{note}
        </if>
    </set>
    where role_no = #{roleNo}
</update>
```

##### foreach元素

foreach元素是一个循环语句，它的作用是遍历集合，它能够很好地支持数组和List、Set接口的集合，对此提供遍历功能。

```xml
<select id="findUserBySex" resultMap="user">
    select * from t_role where role_no in 
    <foreach item="roleNo" index="index" collection="roleNoList"
             open="(" separator=", " close=")">
        #{roleNo}
    </foreach>
</select>
```

- collection配置的roleNoList是传递进来的参数名称
- item配置的是循环中当前的元素
- index配置的是当前元素在集合的位置下标
- open和close配置的是以什么符号将这些集合元素包装起来
- separator是各个元素的间隔符

##### bind元素

bind元素的作用是OGNL表达式去自定义一个上下文变量，这样更方便使用

```xml
<select id="findRole" parameterType="string" resultType="com.xxx.xxxx.bean.RoleBean">
	<bind name="pattern" value="'%' + _parameter + '%'" />
    select id, role_name as roleName, create_date as createDate, end_date as endFlag, end_flag as endFlag, note from t_role where role_name like #{pattern}
</select>
```

### MyBatis的解析和运行原理

MyBatis的运行过程分为两大部：

1. 读取配置文件到Configuration对象
2. SqlSession的执行过程

#### 构建SqlSessionFactory的过程

MyBatis采用了Builder模式去创建SqlSessionFactory，在实际中可以通过SqlSessionFactoryBuilder去创建，其构建可以分为两部：

1. 通过org.apache.ibatis.builder.xml.XMLConfigBuilder解析配置的XML文件，读出所配置的参数，并将读取的内容存入org.apache.ibatis.session.Configuration类对象中。而Configuration采用的是单例模式，几乎所有的MyBatis配置内容都会存放在这个单例对象中，以便后续将这些内容读出。
2. 使用Configuration对象去创建SqlSessionFactory。

MyBatis提供了一个默认的实现类org.apache.ibatis.session.defaults.DefaultSqlSessionFactory.

这种创建方式就是Builder模式，对于复杂的对象而言，使用构造参数很难实现。这时使用一个类Configuration作为统领，一步步构建所需的内容，然后通过它去创建最终的对象SqlSessionFactory。

##### 构建Configuration

Configuration的作用：

1. 读入配置文件，包括基础配置的XML和映射器的XML
2. 初始化一些基础配置，比如MyBatis的别名等，一些重要的类对象（比如插件、映射器、Object工厂、typeHandlers对象等）
3. 提供单例，为后续创建SessionFactory服务，提供配置的参数
4. 执行一些重要的初始化方法

Configuration是通过XMLConfigBuilder去构建的，首先会读出所有XML的配置信息，然后把它们解析并保存在Configuration单例中，它会做如下初始化：

- properties全局参数
- typeAliases别名
- Plugins插件
- objectFactory对象工厂
- objectWrapperFactory对象包装工厂
- reflectionFactory反射工厂
- settings环境设置
- environments数据库环境
- databaseIdProvider数据库标识
- typeHanders类型转换器
- Mappers映射器

##### 构建映射器的内部组成

当XMLConfigBuilder解析XML时，会将每一个SQL和其他配置的内容保存起来。MyBatis中一条SQL和它相关的配置信息是由3个部分组成，分别是MappedStatement，SqlSource和BoundSql。

MappedStatement的作用是保存一个映射器节点（select|insert|update|delete）的内容。它是一个类，包括许多我们配置的SQL、SQL的id、缓存信息、resultMap、parameterType、resultType、resultMap、languageDriver等重要配置内容。

SqlSource是MappedStatement的一个属性。它是一个接口，有三个重要的实现类：DynamicSqlSource、ProviderSqlSource、RawSqlSource和StaticSqlSource。它是提供BoundSql对象的地方，它是根据上下文和参数解析生成需要的Sql，这个接口只定义了一个接口方法——getBoundSql，使用它就可以得到一个BoundSql对象。

BoundSql是一个结果对象，也就是SqlSource通过对SQL和参数的联合解析得到的SQL和参数，它是建立SQL和参数的地方，它有三个常用的属性——sql、parameterObject、parameterMappings。

SqlSource是一个接口，它的主要作用是根据参数和其他的规则组装Sql。对于最终的参数和SQL都反映在BoundSql类对象上，在插件中往往需要拿到它进而可以拿到当前运行SQL和参数，从而对运行过程做出必要的修改，来满足特殊的需求。

BoundSql会提供3个主要的属性：parameterMappings、parameterObject和sql：

parameterObject为参数本身，可以传递简单对象、POJO或者Map、@Param注解的参数

- ​	传递多个参数，如果没有@Param注解，那么MyBatis就会把parameterObject变为一个Map<String, Object>对象，在编写时可以用#{param1}或者#{1}去引用第一个参数；
- ​	使用@Param注解，MyBatis会把parameterObject也变为一个Map<String, Object>对象，但会把数字键值换成@Param注解键值

parameterMappings是一个List，它的每个元素都是ParameterMapping对象。对象会描述参数。参数包括属性名称、表达式、javaType、jdbcType、typeHandler等重要信息。

sql属性是书写在映射器里面的一条被sqlSource解析后的sql。大部分时候无需修改它，只是在使用插件时可以根据需要进行改写。

##### 构建SqlSessionFactory对象

```java
SqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

#### SqlSession运行过程

##### 映射器的动态代理

```java
RoleMapper roleMapper = sqlSession.getMapper(RoleMapper.class);

public class DefaultSqlSession implements SqlSession {
    // ......
    @Override
    public <T> getMapper(class<T> type) {
        return configuration.<T>getMapper(type, this);
    }
    // ......
}
```

用到了Configuration对象的getMapper方法，来获取对应的接口对象的：

```java
// 又运用了映射器的注册器MapperRegistry来回去对应的接口对象
public <T> getMapper(class<T> type, SqlSession sqlSession) {
	return maperRegistry.getMapper(type, sqlSession);
}

public class MapperRegistry {
    // ......
    @SuppressWarnings("unchecked")
    public <T> T getMapper(class<T> type, SqlSession sqlSession) {
        // 获得对应接口的MapperProxyFactory
        final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>)knownMappers.get(type);
        if (mapperProxyFactory == null)
            throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
        try {
            // 返回代理对象实例
            return mapperProxyFactory.newInstance(sqlSession);
        } catch(Exception e) {
            throw new BindingException("Error getting mapper instance.Cause: " +e, e);
        }
    }
    // ......
}
```

使用MapperProxyFactory来生成一个代理实例。

```Java
public class MapperProxyFactory<T> {
    private final Class<T> mapperInterface;
    private final Map<Method, MapperMethod> methodCache = new ConcurrentHashMap();

    public MapperProxyFactory(Class<T> mapperInterface) {
        this.mapperInterface = mapperInterface;
    }

    public Class<T> getMapperInterface() {
        return this.mapperInterface;
    }

    public Map<Method, MapperMethod> getMethodCache() {
        return this.methodCache;
    }


    protected T newInstance(MapperProxy<T> mapperProxy) {
        // 采用JDK动态代理的手段
        return Proxy.newProxyInstance(this.mapperInterface.getClassLoader(), new Class[]{this.mapperInterface}, mapperProxy);
    }

    public T newInstance(SqlSession sqlSession) {
        // 代理的方法被放到了MapperProxy类中
        MapperProxy<T> mapperProxy = new MapperProxy(sqlSession, this.mapperInterface, this.methodCache);
        return this.newInstance(mapperProxy);
    }
}
```

MapperProxy的代码如下：

```Java
public class MapperProxy<T> implements InvocationHandler, Serializable {
    private static final long serialVersionUID = -6424540398559729838L;
    private final SqlSession sqlSession;
    private final Class<T> mapperInterface;
    private final Map<Method, MapperMethod> methodCache;

    public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, MapperMethod> methodCache) {
        this.sqlSession = sqlSession;
        this.mapperInterface = mapperInterface;
        this.methodCache = methodCache;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            // 判断Mapper是否是一个类，这里Mapper是接口
            if (Object.class.equals(method.getDeclaringClass())) {
                return method.invoke(this, args);
            }

            if (this.isDefaultMethod(method)) {
                return this.invokeDefaultMethod(proxy, method, args);
            }
        } catch (Throwable var5) {
            throw ExceptionUtil.unwrapThrowable(var5);
        }
		// 生成MapperMethod对象
        MapperMethod mapperMethod = this.cachedMapperMethod(method);
        return mapperMethod.execute(this.sqlSession, args);
    }
    // ......
}
```

Mapper的XML文件的命名空间对应的是这个接口的全限定名，而方法就是那条Sql的id，这样MyBatis就可以根据全路径和方法名，将其和代理对象绑定起来。通过动态代理技术，让接口运行起来，然后采用命令模式。最后使用SqlSession接口的方法使它能够执行对应的SQL。

##### SqlSession下的四大对象

映射器就是一个动态代理进入到了MapperMethod的execute方法，然后它经过简单的判断就进入了SqlSession的delete、update、insert、select等方法。

SqlSession的执行过程是通过Executor、StatementHamdler、ParameterHandler和ResultSetHandler来完成数据库操作和结果返回的。

- Executor代表执行器，由它调度StatementHandler、ParameterHandler和ResultSetHandler来执行对应的SQL。
- StatementHandler的作用是使用数据库的Statement（PreparedStatement）执行操作。许多重要的插件都是通过拦截它来实现的。
- ParameterHandler用来处理SQL参数
- ResultSetHandler是进行数据集（ResultSet）的封装返回处理的。

**Executor——执行器**

SqlSession其实是一个门面，真正干活的是executor，它是真正执行Java和数据库交互的对象。MyBatis由三种执行器，可以在配置文件中进行选择：

SIMPLE——默认执行器

REUSE——一种能执行重用预处理语句的执行器

BATCH——执行器重用语句和批量更新，批量专用的执行器

执行器提供了我查询（query）、更新（update）方法和相关事务方法。

```Java
	public Executor newExecutor(Transaction transaction) {
        return this.newExecutor(transaction, this.defaultExecutorType);
    }

    public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
        executorType = executorType == null ? this.defaultExecutorType : executorType;
        executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
        Object executor;
        // 根据配置类型去确定需要创建哪种Executor
        if (ExecutorType.BATCH == executorType) {
            executor = new BatchExecutor(this, transaction);
        } else if (ExecutorType.REUSE == executorType) {
            executor = new ReuseExecutor(this, transaction);
        } else {
            executor = new SimpleExecutor(this, transaction);
        }

        // 是否允许缓存，用CacheExecutor来包装executor
        if (this.cacheEnabled) {
            executor = new CachingExecutor((Executor)executor);
        }
		
        // 插件处理
        Executor executor = (Executor)this.interceptorChain.pluginAll(executor);
        return executor;
    }	
```

这里涉及到了插件，它将构建一层层的动态代理对象，我们可以在执行真实的executor方法之前执行我们定义的操作，这就是插件的原理。

接下来看看execute方法内部时怎么处理的：

```java
public class SimpleExecutor extends BaseExecutor {
    public SimpleExecutor(Configuration configuration, Transaction transaction) {
        super(configuration, transaction);
    }
    
    public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
        Statement stmt = null;

        List var9;
        try {
            Configuration configuration = ms.getConfiguration();
            // 通过configuration创建StatementHandler
            StatementHandler handler = configuration.newStatementHandler(this.wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
            // 对sql编译和参数进行初始化
            stmt = this.prepareStatement(handler, ms.getStatementLog());
            // 传递resultHandler
            var9 = handler.query(stmt, resultHandler);
        } finally {
            this.closeStatement(stmt);
        }

        return var9;
    }
    
    private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
        Connection connection = this.getConnection(statementLog);
        // sql编译和参数初始化
        Statement stmt = handler.prepare(connection, this.transaction.getTimeout());
        handler.parameterize(stmt);
        return stmt;
    }
}
```

它调用了StatementHandler的prepare()进行了预编译和基础的设置，然后通过StatementHandler的parameterize()来设置参数，最后调用StatementHandler的query方法，把ResultHandler传递进去，使用它组织结果返回给调用者来完成一次查询。

**StatementHandle——数据库会话器**

专门处理数据库会话的

```Java
// 为Configuration对象的方法
public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    // RoutingStatementHandler通过适配模式来找到对应的StatementHandler来执行的
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    StatementHandler statementHandler = (StatementHandler)this.interceptorChain.pluginAll(statementHandler);
    return statementHandler;
}

public class RoutingStatementHandler implements StatementHandler {
    private final StatementHandler delegate;

    public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
        // 它会根据上下文环境决定创建哪个具体的StatementHanlder对象实例
        // delegate时对象的适配器，为了给三个接口对象的使用提供一个统一且简易的适配器。
        switch(ms.getStatementType()) {
        case STATEMENT:
            this.delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        case PREPARED:
            this.delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        case CALLABLE:
            this.delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
            break;
        default:
            throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
        }

    }
}
```

RoutingStatementHandler提供了三种StatementHandler：SimpleStatementHandler、PrepareStatementHandler和CallableStatementHandler，对应着JDBC的Statement、PrepareStatement和CallableStatement（存储过程处理）

上文的SimpleExecutor的prepareStatement方法有调用StatementHandler的prepare方法：

```Java
// BaseStatementHandler的prepare方法
public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
        ErrorContext.instance().sql(this.boundSql.getSql());
        Statement statement = null;

        try {
            // 对SQL进行预编译，然后做一些基础配置,比如超时、获取最大行数等设置
            statement = this.instantiateStatement(connection);
            this.setStatementTimeout(statement, transactionTimeout);
            this.setFetchSize(statement);
            return statement;
        } catch (SQLException var5) {
            this.closeStatement(statement);
            throw var5;
        } catch (Exception var6) {
            this.closeStatement(statement);
            throw new ExecutorException("Error preparing statement.  Cause: " + var6, var6);
        }
    }
}

// BaseStatementHandler没这方法
public class PreparedStatementHandler extends BaseStatementHandler {
    public PreparedStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
        super(executor, mappedStatement, parameter, rowBounds, resultHandler, boundSql);
    }
    
    public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
        PreparedStatement ps = (PreparedStatement)statement;
        // 直接执行
        ps.execute();
        return this.resultSetHandler.handleResultSets(ps);
    }
}
```

一条查询SQL的执行过程：Executor先调用StatementHandler的prepare方法预编译SQL，同时设置一些基本运行的参数。然后使用parameterize()方法启动ParameterHandler设置参数，完成预编译，执行查询，update()也是这样的。如果是查询，MyBatis会使用ResultSetHandler封装结果返回给调用者。

**ParameterHandler——参数处理器**

MyBatis通过ParameterHandler对预编译语句进行参数设置，它的任务是完成对预编译参数的设置。接口定义如下：

```Java
public interface ParameterHandler {
    // 返回参数对象
    Object getParameterObject();
    // 设置预编译SQL语句的参数
    void setParameters(PreparedStatement ps) throws SQLException;
}
```

在DefaultParameterHandler的setParameters函数中，它从parameterObject对象中取到参数，然后使用typeHandler转换参数，如果有设置，它会根据签名注册的typeHandler对参数进行处理。而typeHandler也是在MyBatis初始化时，注册在Configuration里面的，需要时直接拿来用。

**ResultSetHandler——结果处理器**

ResultSetHandler的接口定义：

```Java
public interface ResultSetHandler {
    <E> List<E> handleResultSets(Statement stmt) throws SQLException;
    void handleOutputParameters(CallableStatement cs) throws SQLException;
}
```

handleOutputParameters()方法处理存储过程输出参数的。

![内部运行图如下](F:\mycode\knowledgeArrangement\web\executor.png)

### 插件

使用插件，就必须实现接口Interceptor，定义如下：

```java
public interface Interceptor {
    Object intercept(Invocation invocation) throws Throwable;
    Object plugin(Object target);
    void setProperties(Properties properties);
}
```

intercept： 直接覆盖拦截对象原有的方法，因为它是插件的核心方法。参数Invocation对象，通过它可以反射调度原来对象的方法。

plugin：target是被拦截对象，它的作用是给被拦截对象生成一个代理对象，并返回它。在MyBatis中，它提供了org.apache.ibatis.plugin.Plugin中的wrap静态方法生成代理对象。

setProperties：在plugin元素中配置所需参数，在插件初始化时被调用一次，然后把插件对象存入到配置中，以便后面使用

这样的模式称为模板（template）模式。

#### 插件的初始化

插件的初始化是在MyBatis初始化时完成的。在解析配置文件时，在MyBatis的上下文初始化过程中，就开始读入插件节点和配置的参数，同时用反射技术生成对应的插件实例，然后调用插件方法中的setProperties方法，设置我们配置的参数，将插件实例保存到配置对象中，以便读取和使用。

##### 插件的代理和反射设计

插件用的是责任链模式。MyBatis中的责任链是由interceptorChain去定义的

```java
executor = (Executor)interceptorChain.pluginAll(executor);

public Object pluginAll(Object target) {
    // 生成代理对象，一层套一层
    for (Interceptor interceptor : interceptors) {
        target = interceptor.plugin(target);
    }
    return target;
}
```

plugin方法是生成代理对象的方法，它是从Configuration对象中取出插件的。

有多少个拦截器就生成多少个代理对象。

MyBatis中提供了一个常用的工具类，用来生成代理对象，它便是Plugin类。Plugin类实现了InvocationHandler接口，采用的是JDK的动态代理

```Java
public class Plugin implements InvocationHandler {
    private final Object target;
    private final Interceptor interceptor;
    private final Map<Class<?>, Set<Method>> signatureMap;

    private Plugin(Object target, Interceptor interceptor, Map<Class<?>, Set<Method>> signatureMap) {
        this.target = target;
        this.interceptor = interceptor;
        this.signatureMap = signatbuzuureMap;
    }

    // 生成动态代理对象
    public static Object wrap(Object target, Interceptor interceptor) {
        Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
        Class<?> type = target.getClass();
        Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
        return interfaces.length > 0 ? Proxy.newProxyInstance(type.getClassLoader(), interfaces, new Plugin(target, interceptor, signatureMap)) : target;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            Set<Method> methods = (Set)this.signatureMap.get(method.getDeclaringClass());
            // 如果存在签名的拦截方法，插件的intercept方法就会在这里调用，如果不存在将直接反射调度要执行的方法
            return methods != null && methods.contains(method) ? this.interceptor.intercept(new Invocation(this.target, method, args)) : method.invoke(this.target, args);
        } catch (Exception var5) {
            throw ExceptionUtil.unwrapThrowable(var5);
        }
    }
}

// MyBatis把被代理方法、反射方法及其参数，都传递给Invocation类的构造方法
public class Invocation {
    private final Object target;
    private final Method method;
    private final Object[] args;

    public Invocation(Object target, Method method, Object[] args) {
        this.target = target;
        this.method = method;
        this.args = args;
    }

    public Object getTarget() {
        return this.target;
    }

    public Method getMethod() {
        return this.method;
    }

    public Object[] getArgs() {
        return this.args;
    }

    public Object proceed() throws InvocationTargetException, IllegalAccessException {
        return this.method.invoke(this.target, this.args);
    }
}
```

Invocation对象的proceed方法就是通过反射的方式调度被代理对象的真实方法的。

注意多个插件的情况，由于使用了职责链模式，所以它首先从最后一个插件开始，先执行其proceed方法之前的代码，然后进入下一个插件proceed方法之前的代码。以此类推知道非代理对象真实方法的调用。然后依次 开始第一个插件proceed方法后的代码，知道最后一个插件proceed方法后的代码。

#### MetaObject

它可以有效读取或者修改一些重要对象的属性。在MyBatis中，四大对象提供的public设置参数的方法很少，难以通过其自身得到相关的属性信息，但是有了MetaObject这个工具类就可以通过其他的技术手段来读取或者修改这些重要对象的属性。

MetaObject有3个方法：

- SystemMetaObject.forObject(Object obj)：获取包装对象；
- Object getValue(String name)方法用于获取对象属性值，支持OGNL；
- void setValue(String name, Object value)方法用于修改对象属性值，支持OGNL。

在插件下修改运行参数：

```Java
StatementHandler statementHandler = (StatementHandler) invocation.getTarget();
MetaObject metaStatementHandler = SystemMetaObject.forObject(statementHandler);
// 进行绑定
// 分离代理对象链（由于目标类可能被多个插件拦截，从而形成多次代理，通过循环可以分离出最原始的目标类）
while (metaStatementHandler.hasGetter("h")) {
    Object object = metaStatementHandler.getValue("h");
    metaStatementHandler = SystemMethodObject.forObject(object);
}

String sql = (String) metaStatementHandler.getValue("delegate.boundSql.sql");
// 判断SQL是否是select语句，如果不是select语句，则不需要处理
// 如果是，则修改它，最多返回1000行，这里用的是MySQL数据库，其他数据库要改写成其他的。
if (sql != null && sql.toLowerCase().trim().indexOf("select") == 0) {
    sql = "select * from (" + sql + ") $_$limit_$table_limit 1000";
    metaStatementHandler.setValues("delegate.boundSql.sql", sql);
}
```

#### 插件开发过程

1. **确定需要拦截的签名**

   Executor是执行SQL的全过程，包括组装参数、组装结果集返回和执行SQL过程。都可以拦截，一般用的不太多。

   StatementHandler是执行SQL的过程，我们可以重写执行SQL的过程，它是最常用的拦截对象。

   ParameterHandler主要拦截执行SQL的参数组装，我们可以重写组装参数规则。

   ResultSetHandler用于拦截执行结果的组装，我们可以重写组装结果规则。

2. **拦截方法和参数**

   拦截器的设计，定义插件的签名

   ```Java
   // @Intercepts说明它是一个拦截器。@Signature是注册拦截器签名的地方，
   // type可以是四大对象中的一个，method代表要拦截的四大对象的某一种接口方法，
   // 而args则标识该方法的参数。
   @Intercepts({
       @Signature(type=StatementHandler.class, 
                  method="prepare",
                  args={Connection.class, Integer})})
   public class MyPlugin implements Interceptor {
       //.......
   }
   ```

   