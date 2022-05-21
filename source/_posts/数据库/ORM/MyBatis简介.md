---
title: MyBatis简介
date: 2022/7/15 20:46:25
tags:
- ORM
- MyBatis
categories:
- [ORM, MyBatis]
description: MyBatis简介
---

# Mybatis的基本使用

## Mybatis的基本配置

### 主配置SqlConfigMapper.xml

```xml

<configuration>
    <!--1 配置properties文件，这通过配置文件的方式动态修改数据库连接信息-->
    <properties resource="jdbcConfig.properties"></properties>
    <typeAliases>
        <!--2 为resultType，parameterType配置别名，不用写全限定类名了 -->
        <package name="com.itheima.domain"></package>
    </typeAliases>

    <!--配置参数-->
    <settings>
        <!--开启Mybatis支持延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>

    <!-- 开启二级缓存 -->
    <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>

    <!--3 配置环境-->
    <environments default="mysql">
        <!-- 配置mysql的环境-->
        <environment id="mysql">
            <!-- 配置事务 -->
            <transactionManager type="JDBC"></transactionManager>
            <!--配置连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"></property>
                <property name="url" value="${jdbc.url}"></property>
                <property name="username" value="${jdbc.username}"></property>
                <property name="password" value="${jdbc.password}"></property>
            </dataSource>
        </environment>
    </environments>
    <!--4 配置映射文件的位置 -->
    <mappers>
        <!--下面三个只能选一个，一般直接配置包路径-->
        <!--配置包路径-->
        <package name="com.itheima.dao"></package>
        <!--配置指定xml文件路径-->
        <mapper resource=""/>
        <!--配置指定的dao接口路径-->
        <mapper class=""/>
    </mappers>
</configuration>
```

### Mapper配置文件

```xml
<!--用于指定映射哪一个接口-->
<mapper namespace="com.itheima.dao.IUserDao">
    <!--参数是简单的基本类型（包括string）,根据id查询用户-->
    <select id="findById" parameterType="INT" resultMap="userMap">
        <!--这里就是方法的参数名-->
        select * from user where id = #{uid}
    </select>

    <!--参数是POJO对象，自动解析pojo的属性,比如下面方法参数是QueryVo，有属性user，但是user也是obj，user有userName属性-->
    <!-- 根据queryVo的条件查询用户 -->
    <select id="findUserByVo" parameterType="com.itheima.domain.QueryVo" resultMap="userMap">
        <!--这里就是方法参数的属性（属性自己还有userName的属性）-->
        select * from user where username like #{user.userName}
    </select>
</mapper>
```

## Mybatis的常用配置

### SQL字段和实体属性不对应

1. SQL使用别名，让字段和实体属性对应
2. 使用`<resultMap>`标签配置对应关系(除此之外还可配置**一对多**和**一对一**关系)

```xml
<!--配置 查询结果的列名和实体类的属性名的对应关系 -->
<resultMap id="userMap" type="uSeR">
    <!--主键字段的对应 -->
    <id property="userId" column="id"></id>
    <!--非主键字段的对应-->
    <result property="userName" column="username"></result>

    <result property="userAddress" column="address"></result>

    <result property="userSex" column="sex"></result>

    <result property="userBirthday" column="birthday"></result>
</resultMap>
```

### 一对一

```xml
<!--定义封装account和user的resultMap -->
<resultMap id="accountUserMap" type="account">
    <id property="id" column="aid"></id>
    <result property="uid" column="uid"></result>
    <result property="money" column="money"></result>
    <!--一对一的关系映射：配置封装user的内容-->
    <association property="user" column="uid" javaType="user">
        <id property="id" column="id"></id>
        <result column="username" property="username"></result>
        <result column="address" property="address"></result>
        <result column="sex" property="sex"></result>
        <result column="birthday" property="birthday"></result>
    </association>
</resultMap>

<!--查询所有，执行的SQL-->
<select id="findAll" resultMap="accountUserMap">
    select u.*,a.id as aid,a.uid,a.money from account a , user u where u.id = a.uid;
</select>
```

### 一对多

```xml
<!--定义User的resultMap-->
<resultMap id="userAccountMap" type="user">
    <id property="id" column="id"></id>
    <result property="username" column="username"></result>
    <result property="address" column="address"></result>
    <result property="sex" column="sex"></result>
    <result property="birthday" column="birthday"></result>
    <!--配置user对象中accounts集合的映射 -->
    <collection property="accounts" ofType="account">
        <id column="aid" property="id"></id>
        <result column="uid" property="uid"></result>
        <result column="money" property="money"></result>
    </collection>
</resultMap>

<!--查询所有，执行的SQL-->
<select id="findAll" resultMap="userAccountMap">
    select * from user u left join account a on u.id = a.uid
</select>
```

# 延迟加载

**延迟加载**：就是在需要用到数据时才进行加载，不需要用到数据时就不加载数据。延迟加载也称懒加载

1. 好处：先从单表查询，需要时再从关联表去关联查询，大大提高数据库性能，因为查询单表要比关联查询多张表速度要快。
2. 坏处：因为只有当需要用到数据时，才会进行数据库查询，这样在大批量数据查询时，因为查询工作也要消耗时间，所以可能造成用户等待时间变长，造成用户体验下降。

## 配置延迟加载

### 开启延迟加载(主配置文件)

```xml
<!--配置参数-->
<settings>
    <!--开启Mybatis支持延迟加载-->
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

### select属性

一对多，多对多中`<collection>`和`<association>`标签的属性`select`（**重点**) 是实现懒加载的基础：由于存在一对多，多对多的关系；才可能需要延迟加载。 `<collection>`
和`<association>`标签中需要指定`column`，`javaType`（`ofType`），`select`根据`column`中的参数来查询对应的`javaType`(`ofType`)。 `select`
中的是查询`javaType`（`ofType`）的方法

## 使用延迟加载

### 一对一

```xml
<!-- 定义封装account的resultMap -->
<resultMap id="accountUserMap" type="account">
    <id property="id" column="id"></id>
    <result property="uid" column="uid"></result>
    <result property="money" column="money"></result>
    <!-- 一对一的关系映射：配置封装user的内容
         select属性指定的内容：查询用户的唯一标识：
         column属性指定的内容：用户根据id查询时，所需要的参数的值
    -->
    <!-- 如果是同一个接口的方法select可以简写为findAccountByUid，qid是变量名(select对应查询里面的变量)，id是主表的字段 -->
    <association
            property="user"
            column="uid"
            javaType="user"
            select="com.itheima.dao.IUserDao.findById"/>
</resultMap>
<!-- 查询所有 -->
<select id="findAll" resultMap="accountUserMap">
    select * from account
</select>
<!-- 根据用户id查询账户列表 -->
<select id="findAccountByUid" resultType="account">
    select * from account where uid = #{uid}
</select>
```

### 一对多

```xml
<!-- 定义User的resultMap-->
<resultMap id="userAccountMap" type="user">
    <id property="id" column="id"></id>
    <result property="username" column="username"></result>
    <result property="address" column="address"></result>
    <result property="sex" column="sex"></result>
    <result property="birthday" column="birthday"></result>
    <!-- 配置user对象中accounts集合的映射 -->
    <!-- 如果是同一个接口的方法select可以简写为findAccountByUid，qid是变量名(select对应查询里面的变量)，id是主表的字段 -->
    <collection
            property="accounts"
            column="{qid=id,sort=sort}"
            ofType="account"
            select="com.study.dao.IAccountDao.findAccountByUid"/>
</resultMap>
<!-- 查询所有 -->
<select id="findAll" resultMap="userAccountMap">
    select * from user
</select>
<!-- 根据id查询用户 -->
<select id="findById" parameterType="INT" resultType="user">
    select * from user where id = #{uid}
</select>
```

# 使用Mybatis

## SQL抽取

```xml
<!--抽取sql-->
<sql id="defaultUser">
    select * from user
</sql>
<!--引用sql-->
<select id="findAll" resultMap="userMap">
    <include refid="defaultUser"/>
</select>
```

## 获取主键自增

```xml
<!-- 方案一：<selectKey>标签内部使用select last_insert_id() -->
<insert id="saveUser" parameterType="user">
    <!-- 配置插入操作后，获取插入数据的id -->
    <selectKey keyProperty="userId" keyColumn="id" resultType="int" order="AFTER">
        select last_insert_id();
    </selectKey>
    insert into user(username,address,sex,birthday)values(#{userName},#{userAddress},#{userSex},#{userBirthday});
</insert>

<!-- 方案二：使用标签内属性(最简单的)：usegeneratedkeys="true" "keyproperty"="id" keyColum="id"-->
<insert id="saveUser" parameterType="user" usegeneratedkeys="true" keyproperty="id" keyColum="id">
    insert into user(username,address,sex,birthday)values(#{userName},#{userAddress},#{userSex},#{userBirthday});
</insert>
```

## #{}和${}区别

1. `#{}`：表示一个占位符号通过`#{}`可以实现`preparedStatement`向占位符中设置值，自动进行**类型转换**，`#{}`可以有效防止`sql`注入。 `#{}`可以接收简单类型值或`POJO`属性值。
   如果`parameterType`传输单个简单类型值，`#{}`括号中可以是`value`或其它名称。
2. `${}`：表示拼接`sql`串通过`${}`可以将`parameterType` 传入的内容拼接在`sql`中且不进行**类型转换**，`${}`可以接收简单类型值或`POJO`属性值，如果`parameterTyp`
   传输单个简单类型值，`${}`括号中只能是`value`

## 动态SQL

```xml

<select id="findUserByCondition" resultMap="userMap" parameterType="user">
    select * from user
    <where>
        <!--内部sql以and开头，一定要注意where标签会自动判断的and是需要去掉-->
        <if test="userName != null">
            and username = #{userName}
        </if>
        <if test="userSex != null">
            and sex = #{userSex}
        </if>
    </where>
</select>
        <!-- in范围查询使用foreach,固定写法-->
<select id="findUserInIds" resultMap="userMap" parameterType="queryvo">
    <include refid="defaultUser"></include>
    <where>
        <!--open第1种写法-->
        <if test="ids != null and ids.size()>0">
            <foreach collection="ids" open="and id in (" close=")" item="uid" index="index" separator=",">
                #{uid}
            </foreach>
        </if>
        <!--open第2种写法-->
        <if test="ids != null and ids.size()>0">、
            and id in
            <foreach collection="ids" open="(" close=")" item="uid" index="index" separator=",">
                #{uid}
            </foreach>
        </if>
    </where>
</select>
```

## CRUD操作

```java
public class Test {
    private InputStream in;
    private SqlSession sqlSession;
    private IUserDao userDao;

    @Before//用于在测试方法执行之前执行
    public void init() throws Exception {
        //1.读取配置文件，生成字节输入流
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.获取SqlSessionFactory
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        //3.获取SqlSession对象
        sqlSession = factory.openSession(true);
        //4.获取dao的代理对象
        userDao = sqlSession.getMapper(IUserDao.class);
    }

    @After
    //用于在测试方法执行之后执行
    public void destroy() throws Exception {
        //提交事务
        // sqlSession.commit();
        //6.释放资源
        sqlSession.close();
        in.close();
    }

    // 测试查询所有，执行的查询逻辑
    @Test
    public void testFindAll() {
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println("-----每个用户的信息------");
            System.out.println(user);
            System.out.println(user.getAccounts());
        }
    }
}
```

## Spring整合Mybatis

```xml
<!--配置SqlSessionFactoryBean-->
<bean id="sqlSessionFactory" class="sqlSessionFactoryBean">
    <property name="datasource" ref="datasource">
</bean>
<!--配置扫描Dao的包-->
<bean id="mapperscanner" class="MapperScannerConfiguer">
    <property name="package" ref="com.duhao.dao">
</bean>
```

# Mybatis缓存机制

`Mybatis`和大多数持久化框架一样，提供了缓存策略，通过缓存策略来减少数据库的查询次数，提高性能。`Mybatis`中缓存分为一级缓存，二级缓存。

![f91f1bdd-5cdc-45fa-b133-05a7cbdadf73](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/mybatis%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6.png)

## 一级缓存(sqlSession级别)

一级缓存是`SqlSession`范围的缓存，`SqlSession`的内部有`HashMap`保存缓存数据，当调用`SqlSession`的`update,insert,delete`等操作或者`commit(),close()`
等方法时，就会清空一级缓存。

![b91cda7d-4375-4250-8948-331597dc2447](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/mybatis%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98.png)

1. 第一次发起查询用户`id`为1的用户信息，先去找缓存中是否有`id`为1的用户信息，如果没有，从数据库查询用户信息。得到用户信息，将用户信息存储到一级缓存中。
2. 如果 `SqlSession` 去执行 `commit` 操作（执行插入、更新、删除），清空 `SqlSession` 中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。
3. 第二次发起查询用户 `id` 为1的用户信息，先去找缓存中是否有 `id` 为1的用户信息，缓存中有，直接从缓存中获取用户信息。

## 一级缓存清空

1. `sqlSession.commit()`和`sqlSession.close()`
2. 调用 `mapper` 接口的`insert,update,delete `等操作的方法
3. `sqlSession.clearCache()`手动清理缓存

## 二级缓存(Mapper级别)

二级缓存是`mapper`映射级别的缓存（缓存在`sqlSessionFactory`中），多个 `SqlSession` 去操作同 一个`Mapper` 映射的 `sql` 语句，多个`SqlSession`
可以共用二级缓存，二级缓存是跨`SqlSession`的。

![53e71055-2a2e-4585-9c65-befe0ee49fca](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/myabtis%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98.png)

1. 首先开启`Mybatis`的二级缓存。
2. `sqlSession1`去查询用户信息，查询到用户信息会将查询数据存储到二级缓存中。
3. 如果`SqlSession3`去执行相同 `mapper`映射下`sql`，执行`commit`提交，将会清空该 `mapper`映射下的二级缓存区域的数据。
4. `sqlSession2`去查询与`sqlSession1`相同的用户信息，首先会去缓存中找是否存在数据，如果存在直接从缓存中取出数据。

## 开启二级缓存

当我们在使用二级缓存时，所缓存的类一定要实现`java.io.Serializable`接口，这种就可以使用序列化方式来保存对象。

```xml
<!--主配置文件-->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

```xml
<!--mapper配置文件-->
<!--开启user支持二级缓存-->
<cache/>

<!--mapper中statement添加userCache属性 -->
<select id="findById" parameterType="INT" resultType="user" useCache="true">
    select * from user where id = #{uid}
</select>
```

# Mybatis注解开发

## 常用注解

1. `@Insert`：实现新增
2. `@Update`：实现更新
3. `@Delete`：实现删除
4. `@Select`：实现查询
5. `@Results`：
    1. 可以与`@Result`一起使用，封装多个结果集，代替的是标签`<resultMap>`
    2. 该注解中可以使用单个`@Result`注解，也可以使用`@Result`集合
    3. `@Result`：实现结果集封装
        1. `@Results(){@Result(),@Result()})`或`@Results(@Result())`
        2. 属性`one`：
            1. 值`@One`注解，替换`<assocation>`
            2. `@One`属性`select`指定用来查询的方法
            3. `@One`属性`fetchType`（加载类型：延迟，即时）会覆盖全局的配置参数`lazyLoadingEnabled`
        3. 属性`many`
            1. 值`@Many`注解，替换`<Collection>`标签
            2. `@Many`属性`select`指定用来查询的方法
            3. `@Many`属性`fetchType`（加载类型：延迟，即时）会覆盖全局的配置参数 `lazyLoadingEnabled`
6. `@ResultMap`：实现引用 `@Results` 定义的字段属性映射关系
7. `@SelectProvider`：实现动态 `SQL` 映射
8. `@CacheNamespace`：实现注解二级缓存的使用

```java
//基于注解实现二级缓存
@CacheNamespace(blocking = true)
public interface IUserDao {
    @Results(id = "userMap", value = {
            @Result(id = true, column = "id", property = "userId"),
            @Result(column = "username", property = "userName"),
            @Result(column = "address", property = "userAddress"),
            @Result(column = "sex", property = "userSex"),
            @Result(column = "birthday", property = "userBirthday"),
            @Result(
            property = "user",
            column = "uid",
            one = @One(
                select = "com.itheima.dao.IUserDao.findById",
                // 延迟加载
                fetchType = FetchType.EAGER)
            ),
            @Result(
                property = "accounts",
                column = "id",
                many = @Many(
                    select = "com.itheima.dao.IAccountDao.findAccountByUid",
                    // 延迟加载
                    fetchType = FetchType.LAZY)
            )
            })
    @Select("select * from user")
    List<User> findAll();

    //定义的@Results可以被其他方法引用
    @Select("select * from user where id=#{id} ")
    @ResultMap("userMap")
    User findById(Integer userId);
}
```