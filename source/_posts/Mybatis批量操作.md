title: Mybatis批量操作
date: 2016-12-14 21:35:38
categories: 技术
tags: MyBatis
---
在项目开发的过程中，一般的操作都是单条数据操作，插入一次提交一次事务；但是往往会有大批量类似的数据需要插入，此时用循环单条插入的方式去操作也可以，但是频繁的请求数据库资源很耗费系统资源，有没有更好的方式呢？

<!--more-->

发现问题后就要尝试查找资料去解决问题，去看 MyBatis 的源码？一大堆不知道从那看起？那就去看Github。众多周知， Mybatis 源码等相关信息是托管在 `Github` 上的，可以看看他的 wiki 里有没有相应的方案，在它的 [FAQ(Frequently Asked Questions)][1]的中有关于批量插入的示例：
首先，创建一个简单的插入语句：

    <insert id="insertName">
      insert into names (name) values (#{value})
    </insert>

然后在 Java 代码中执行批量操作:

    List<String> names = new ArrayList<String>();
    names.add("Fred");
    names.add("Barney");
    names.add("Betty");
    names.add("Wilma");
    
    SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH);
    try {
      NameMapper mapper = sqlSession.getMapper(NameMapper.class);
      for (String name : names) {
        mapper.insertName(name);
      }
      sqlSession.commit();
    } finally {
      sqlSession.close();
    }

示例中的操作是 insert，同理 update、delete 等的操作也一样，要点是事务提交一次。 

目前的数据库有多种，各个数据库批量插入的方式不一样，但是对于单条插入然后去循环的方式是都支持的，所以 MyBatis 做了一个共通的实例，将执行类型设置`ExecutorType.BATCH`,批量去提交事务。

示例中的核心在于创建一个操作批量事物的SqlSession，在纯 MyBatis 中 SqlSession 的获取方式为：

    InputStream in=Resources.getResourceAsStream("applicationContext.xml");  
    SqlSessionFactory factory=new SqlSessionFactoryBuilder().build(in);   
    SqlSession sqlsession=factory.openSession();  

查看 Mybatis 中可看到默创建的事务为简单事务： `ExecutorType.SIMPLE`，批量插入要修改事务类型。

## Java 程序处理

在后续的开发中，MyBatis 常与其他框架一起使用，最常见的是 SSM(Spring+SpringMVC+Mybatis) 的整合，通常对数据库事务的操作就交给Spring 去处理了。此时如果有大量数据需要插入，利用循环去 mapper.inset(entity) 也可以解决问题，但是每执行一次都需要开启一个新的事务，对资源的消耗比较大。所以，要采用其他的更优化的方案。

在 Spring 中不用关心 SqlSession 是如何创建销毁的，在applicationContext.xml中配置响应的 SqlsessionFactory 即可。

    <!-- 配置SqlSessionFactoryBean -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
      <property name="dataSource" ref="dataSource"/>
      <property name="configLocation" value="classpath:mybatis.xml"/>
    </bean>

如果要在 Java 中去控制事务类型，可通过注入 SqlSeesionFactory 的方式显示的获取 SqlSession，

    @Autowired
    private SqlSessionFactoryBean sqlSessionFactory;
    ……
    SqlSession sqlSession=sqlSessionFactory.getObject().openSession(ExecutorType.BATCH);

获取到 SqlSession 之后在仿照示例中的代码写就可以了。

## SQL 拼接处理

另一种方式是通过动态拼接 SQL 的形式去批量插入，这是的主要工作是在数据库那一端进行的。各个数据库批量插入的方式不一样，以下举最常见的两个数据库 MySQL 与 Oracle，其他的数据库欢迎补充。

Dao 中的代码：

    public interface userDao{
      public void insertBatch(@Param("userList") List userList);
    }

### MySQL

MySQL 支持如下类型的 SQL 语句：

    insert into users(username,age) values('zhangsan'，18),('liming',20),('fengyu',23);

上述 SQL 是 MySQL 特有的，可在*.xml 文件中去执行拼接，

    <insert id="insertBatch" parameterType="java.util.List">  
      insert into users(username,age) values
      <foreach collection="userList" item="item" index="index"  separator=",">  
       (#{item.username},#{item.age})
      </foreach>  
    </insert> 

### Oracle

Oracle 不支持 MySQL 那种写法，但是他也有自己的其他语句来实现这样的功能：

Oracle 中可以使用 insert all，insert first ，union all 等语句去执行批量的事务操作。

union all：

    <insert id="insertBatch" parameterType="java.util.List">  
      insert into users(username,age) 
      <foreach collection="freeNumberList" item="item" index="index"  separator="union all">  
        select #{item.username},#{item.age} from dual
      </foreach>  
    </insert> 

insert all：

    <insert id="insertBatch" parameterType="java.util.List">  
      insert all 
      <foreach collection="freeNumberList" item="item" index="index"> 
       into users(username,age) values(#{item.username},#{item.age})
      </foreach>  
      select 1 from dual
    </insert> 

insert first 与 insert all 区别在于insert all 只要符合条件就都插入，insert first有一个条件判断，可以在 sql 中加上 where 条件。

具体使用是使用 Java 代码的方式，还是使用 SQL 拼接的方式，要根据具体的项目定了。

目前还没有做关于这两种方式的批量插入的效率问题，后续再补。



[1]: https://github.com/mybatis/mybatis-3/wiki/FAQ
[2]: http://www.mybatis.org/mybatis-3/zh/dynamic-sql.html

