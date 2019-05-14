---
title: 「置顶」东软睿道实训杂记
date: 2018-05-08 10:00:00
category: "Java"
top: 2
tags: 
    - 东软睿道
    - 实习
    - Java
    - 互联网架构
    - MyBatis
    - Spring IOC
    - Spring AOP
---

# Neuedu
Neuedu (东软睿道)  
实训内容：Java互联网架构基础知识  
2019.05-07 - 至今  
<img src="http://ws1.sinaimg.cn/large/006tNc79ly1g2sta9fzrjj303w027glg.jpg" width="200rpx"/>

# 目录
- [Day 1 - Day 4AM: MyBatis](#-day-1---day-4am:-mybatis)  
- [Day 4PM - Day 5: Spring IOC](#-Day-4PM---Day-5:-Spring-IOC)  
- [Day 6 - Day X: Spring AOP](#Day-6---Day-X:-Spring-AOP) 
- Learning... 

现仍处于学习阶段，随着学习程度的加深会对现有知识有新的理解以及找寻到更好的参考资料，随时会对笔记进行更新。😋  
为了获得更优阅读体验，您可移步至我的博客。  
[东软睿道实训杂记](https://ravenxu.top/2019/05/07/%E4%B8%9C%E8%BD%AF%E7%9D%BF%E9%81%93%E5%AE%9E%E8%AE%AD%E6%9D%82%E8%AE%B0/)

# Day 1 - Day 4AM: MyBatis
[参考代码](https://github.com/Raven98/Neuedu/tree/master/TestMyBatis)  
[MyBatis 英文文档](http://www.mybatis.org/mybatis-3/)  
[MyBatis 中文文档](http://www.mybatis.org/mybatis-3/zh/getting-started.html)  
本文内容很大程度地参考了官方文档，mybatis有中文版本的文档并且极其简洁，推荐阅读
## 1. 了解MyBatis
- 优秀的持久层框架
- 支持定制化 SQL、存储过程以及高级映射
- 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集
- 使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录

<!--more-->

## 2. 传统的JDBC
MyBatis是对JDBC的封装，使用JDBC我们需要以下步骤：

### 2.1. Load driver into memory 注册驱动
```java
Class.forName("com.mysql.jdbc.Driver");
```
### 2.2. Get connection from database 创建connection
```java
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/scott", "root", "root");
```
### 2.3. Create preparedStatement to write a Sql
```java
PreparedStatement stat = conn.prepareStatement("select * from dept where deptno = ?");
```
### 2.4. Replace all ? with actual value
```java
stat.setInt(1, 10);
```
### 2.5 execute sql
```java
ResultSet rs = stat.executeQuery();
```
### 2.6. get the result
```java
if(rs.next())
{
	int deptno = rs.getInt("deptno");
	String dname = rs.getString("dname");
	String loc = rs.getString("loc");
	
	System.out.println(deptno+"\t"+dname+"\t"+loc);
}
```

## 3. 试一试MyBatis
### 3.1. 从 XML 中构建 SqlSessionFactory
每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。
```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
            SqlSessionFactory sqlSessionFactory =
                    new SqlSessionFactoryBuilder().build(inputStream);
```
也可以不从XML而是直接在Java中构建SqlSessionFactory，不做详细讨论。
### 3.2. 从 SqlSessionFactory 中获取 SqlSession
既然有了 SqlSessionFactory，顾名思义，我们就可以从中获得 SqlSession 的实例了。SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句。例如：
```java
SqlSession session = sqlSessionFactory.openSession();
```
```java
Dept d = session.selectOne("com/neuedu/model/DeptMapper.java.selectDept", 20);
System.out.println(d.getDeptno()+"\t"+d.getDname()+"\t"+d.getLoc());
```
诚然，这种方式能够正常工作，并且对于使用旧版本 MyBatis 的用户来说也比较熟悉。不过现在有了一种更简洁的方式 ——使用正确描述每个语句的参数和返回值的接口（比如 DeptMapper.class），你现在不仅可以执行更清晰和类型安全的代码，而且还不用担心易错的字符串字面值以及强制类型转换。  
例如：
```java
DeptMapper mapper = session.getMapper(DeptMapper.class);
Dept d = mapper.selectDept(20);
System.out.println(d.getDeptno()+"\t"+d.getDname()+"\t"+d.getLoc());
```
>新旧方式对比(个人理解)  
![](http://ws3.sinaimg.cn/large/006tNc79ly1g2sifw8z3tj30fz0aujrd.jpg)
旧方式业务代码直接根据(namespace+id)调用XML里面的某一个方法  
新方式将XML和接口双向绑定，业务代码不需要知道XML文件内容，直接调用接口。  
一个显而易见的变化就是旧方式的namespace随便写，而新方式必须为xml文件所在路径  
第二种方法有很多优势，首先它不依赖于字符串字面值，会更安全一点； 其次，如果你的 IDE 有代码补全功能，那么代码补全可以帮你快速选择**已映射的 SQL 语句**。  
这样做的好处在学习初期还不能完全理解，随着学习的加深能逐渐领悟  
### 3.3. 获取查询结果
```java
//to iterate this list
Iterator<Dept> it = d.iterator();
while(it.hasNext())
{
    Dept item = it.next();
    //output its content
    System.out.println(item.getDeptno()+"\t"+item.getDname()+"\t"+item.getLoc());
}
```
## 4. 对比JDBC和MyBatis
直观的代码改变是MyBatis需要一个主XML配置文件，需要根据业务需求写一些mapper XML文件，需要为每个mapper XML文件编写对应的接口文件，需要为数据库内实体编写对应的Java类。  
多了很多文件，但一定有很多好处：  
[参考资料](https://www.cnblogs.com/love-Stefanie/p/6838269.html)
### 4.1. 优化获取和释放  
我们一般在访问数据库时都是通过数据库连接池来操作数据库，数据库连接池有好几种，比如C3P0、DBCP，也可能采用容器本身的JNDI数据库连接池。我们可以通过DataSource进行隔离解耦，我们统一从DataSource里面获取数据库连接，DataSource具体由DBCP实现还是由容器的JNDI实现都可以，所以我们将DataSource的具体实现通过让用户配置来应对变化。
### 4.2. SQL统一管理，对数据库进行存取操作  
我们使用JDBC对数据库进行操作时，SQL查询语句分布在各个Java类中，这样可读性差，不利于维护，当我们修改Java类中的SQL语句时要重新进行编译。  
Mybatis可以把SQL语句放在配置文件中统一进行管理，以后修改配置文件，也不需要重新就行编译部署。
### 4.3. 生成动态SQL语句  
我们在查询中可能需要根据一些属性进行组合查询，比如我们进行商品查询，我们可以根据商品名称进行查询，也可以根据发货地进行查询，或者两者组合查询。如果使用JDBC进行查询，这样就需要写多条SQL语句。  
Mybatis可以在配置文件中通过使用`<if test=””></if>`标签进行SQL语句的拼接，生成动态SQL语句。比如下面这个例子：
```xml
<select id="getCountByInfo" parameterType="User" resultType="int">
        select count(*) from user
        <where>
            <if test="nickname!=null">
                and nickname = #{nickname} 
            </if>
            <if test="email!=null">
                and email = #{email} 
            </if>
        </where>
</select>
```
就是通过昵称或email或者二者的组合查找用户数。  
### 4.4. 能够对结果集进行映射（手动->自动）  
我们在使用JDBC进行查询时，返回一个结果集ResultSet，我们要从结果集中取出结果封装为需要的类型。  
在Mybatis中我们可以设置将结果**直接**映射为自己需要的类型，比如：JavaBean对象、一个Map、一个List等等。
## 5. 对mybatis-config.xml(主配置文件)的配置简化
### 5.1. 使用properties管理变量
数据库driver url username password等
```text
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/scott
username=root
password=root
```
在mybatis-config.xml中导入
```xml
<!--Load db.properties-->
<properties resource="db.properties"></properties>
```
### 5.2. 使用类型别名
resultType省去路径前缀
```xml
<typeAliases>
    <package name="com.neuedu.model.po"/>
</typeAliases>
```
### 5.3. 一次引入所有mapper文件
```xml
<mappers>
    <package name="./mapper"/>
</mappers>
```
不过我在intellij里没一次导入成功，貌似找不到路径，只得挨个导入mapper
```xml
<mappers>
    <mapper resource="mapper/DeptMapper.xml"></mapper>
    <mapper resource="mapper/EmpMapper.xml"></mapper>
    <mapper resource="mapper/ScoresMapper.xml"></mapper>
</mappers>
```
## 6. 使用log.4j日志管理工具
[最详细的Log4J使用教程](https://blog.csdn.net/u013870094/article/details/79518028)
### 6.1. 配置文件
```text
# Global logging configuration
log4j.rootLogger=DEBUG, stdout
# MyBatis logging configuration...
log4j.logger.org.mybatis.example.BlogMapper=TRACE
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```
### 
## 7. 尝试更多sql
1. like语句
2. 更多变量
3. 插入
4. 插入递增字段
5. 插入一个对象instance
6. 指定返回字段
7. 利用hashmap传参
8. update
9. delete
10. 多表查询
    - 返回值不能是po了
    - 在涉及到的表的mapper选一个。一般选主要的一个

## 8. Result Maps
hashmap或者po作为返回值需要字段名与属性名一致，但有时数据库字段的user_id和Java属性的驼峰式userId不一样，需要有一个映射(不用resultmap的话也可以在sql里用别名关键字`as`)
### 8.1. 一对一多表查询
### 8.2. 一对多多表查询
### 8.3. 多对多多表查询
### 8.4. LazyLoading
### 8.5. 利用`<sql>`替换重复sql代码
### 8.6. 字符串替换
因为Preparestatement后sql的结构不变
> \#和$的区别  
> 
> 理解「sql结构与变量」的概念    
> $可以改变sql的结构有sql注入的风险

## 9. 动态SQL
### 9.1. 试一下传统方法
需要大量判断或者多次查询
### 9.2. if标签
省去了判断根据属性的顺序传值的部分代码，但还是需要写不同if的and代码  
利用where + if标签完成
```xml
<select id="getEmpByConditions" resultType="Emp">
    select * from emp
    <where>
        <if test="empno != 0">
            and empno = #{empno}
        </if>
        <if test="ename != null">
            and ename = #{ename}
        </if>
        <if test="job != null">
            and job = #{job}
        </if>
    </where>
</select>
```
### 9.3. choose + when + otherwise (switch case default)
```xml
<select id="getEmpByOneCondition" resultType="Emp">
    select * from emp
    <where>
        <choose>
            <when test="empno != 0">
                and empno = #{empno}
            </when>
            <when test="ename != null">
                and ename = #{ename}
            </when>
            <when test="job != null">
                and job = #{job}
            </when>
        </choose>
    </where>
</select>
```
### 9.4. Dynamic Update
```xml
<update id="updateEmpByCondition">
    update emp
    <set>
        <if test="ename != null">
            ename = #{ename},
        </if>
        <if test="job != null">
            job = #{job},
        </if>
    </set>
    where empno = #{empno}
</update>
```

### 9.5. foreach

### 9.6. 分页 limit
```xml
<select id="getEmpByPage" resultType="Emp">
    select * from emp limit #{index}, #{count}
</select>
```
## 10. MyBatis Cache
mybatis的性能是不如JDBC的，但是它通过cache等操作尽力提升了性能(虽然还是差jdbc很多)
**demo**
```java
public static void queryByPage() {
    SqlSession session = DBUtils.getInstance();
    EmpMapper mapper = session.getMapper(EmpMapper.class);

    List<Emp> emps = mapper.getEmpByPage(0, 10);
    for(Emp e : emps) {
        System.out.println(e.getEmpno() + "\t" + e.getEname() + "\t" + e.getJob());
    }

    // this time, get data from cache
    // by default, data will be cached and shared in one session
    // if you create two different sessions, data can not be shared
    List<Emp> emps2 = mapper.getEmpByPage(0, 10);
    for(Emp e : emps) {
        System.out.println(e.getEmpno() + "\t" + e.getEname() + "\t" + e.getJob());
    }
}
```
如下图所示，同一个session里的同样的sql语句只被执行一次
![](http://ws1.sinaimg.cn/large/006tNc79ly1g2uw4nfuc0j31wk0bkwit.jpg)

## 11. Maven Project
优点
- 在一个位置管理全部依赖包
  - 只需写几行代码，导入方便
  - 自动将依赖的jar一次性下载完毕
  - 发送给别人方便，不需要附带依赖的jar包，接收方可以根据pom.xml自行导入依赖
- 有mvn命令，便于打包发布持续集成自动化构建

## 附1：本单元补充知识点
### 1. iBatis
### 2. Java Bean
### 3. sql.date and utils.date
[java.util.Date和java.sql.Date的区别及应用](https://www.cnblogs.com/IamThat/p/3264234.html)

### 4. SQL注入与Preparestatement
[预处理prepareStatement是怎么防止sql注入漏洞的？](https://www.cnblogs.com/yaochc/p/4957833.html)
## 附2：一点思考
1. MyBatis底层还是对JDBC的封装，查看运行日志可知最终运行的还是sql代码，只不过使用mybatis可以帮助我们更好地管理复杂的数据库访问代码，以更贴近Java本身的设计思想的方式还访问数据库。  
2. XML 接口文件 理解这些文件的作用需要领悟**映射**的概念。  
   一个mapper.mxl对应一个接口文件对应一个数据库的表
3. MyBatis
   MyBatis作为一个框架，还是对现有接口的封装，可以参考轻量级框架[sql2o](https://www.sql2o.org/)的设计思想自行设计一个框架
4. 存储过程的使用场景：阿里巴巴Java开发手册明确写明了禁止使用存储过程，究其原因更多是因为存储过程代码可读性极差、debug困难，对于阿里这样的大企业有其他措施弥补性能；对于东软这样的外包公司，如果是一个**需求明确**的任务还是可以写存储过程的，毕竟因为存储过程在数据库内一次性完成多个操作性能会更好。


# Day 4PM - Day 5: Spring IOC

## 参考资料
- [参考代码(基于XML)](https://github.com/Raven98/Neuedu/tree/master/SpringCore)  
- [参考代码(基于注解)](https://github.com/Raven98/Neuedu/tree/master/testspringannotation)  
- [Spring官方文档](https://docs.spring.io/spring/docs/5.1.6.RELEASE/spring-framework-reference/core.html)  
- [费老师著作](https://www.kancloud.cn/winter1981/spring/543484)

## 1. 了解Spring IOC
### 1.1. IOC: invertion of control
#### Demo for IOC
```java
public class TestService {
    // IOC inversion of control, also called DI(dependency injection)
    private TestDAO testDAO;

    public void setTestDAO(TestDAO testDAO) {
        this.testDAO = testDAO;
    }

    public static void main(String[] args) {
        //get a instance of TestService from Spring
        //ask spring for instance
        //create a Spring container,
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        //get instance from Spring container
        //by default, spring uses singleton mode
        TestService service = (TestService)context.getBean("testService");
        TestService service2 = (TestService)context.getBean("testService");
        TestService service3 = (TestService)context.getBean("testService");

        System.out.println(service);
        System.out.println(service2);
        System.out.println(service3);
    }
}
```

#### Ouput
```text
testspringcore.TestService@6d4b1c02
testspringcore.TestService@6d4b1c02
testspringcore.TestService@6d4b1c02

Process finished with exit code 0
```
IOC的目的是要让Sring来创建我们所需要的对象，不需要通过new建立。我们在applicationContext.xml配置Java bean对象即可被spring所调用。Spring默认使用单例模式。（上面的hash值指向同一个对象）由Spring所创建的对象被放到Spring的容器中。

### 1.2.单例模式 Singleton
> 单例模式与静态类
> 观点一：（单例 ）
单例模式比静态方法有很多优势：
首先，单例可以继承类，实现接口，而静态类不能（可以集成类，但不能集成实例成员）；
其次，单例可以被延迟初始化，静态类一般在第一次加载是初始化；
再次，单例类可以被集成，他的方法可以被覆写；
最后，或许最重要的是，单例类可以被用于多态而无需强迫用户只假定唯一的实例。举个例子，你可能在开始时只写一个配置，但是以后你可能需要支持超过一个配 置集，或者可能需要允许用户从外部从外部文件中加载一个配置对象，或者编写自己的。你的代码不需要关注全局的状态，因此你的代码会更加灵活。
>
>观点二：（静态方法 ） 静态方法中产生的对象，会随着静态方法执行完毕而释放掉，而且执行类中的静态方法时，不会实例化静态方法所在的类。如果是用singleton,   产生的那一个唯一的实例，会一直在内存中，不会被GC清除的(原因是静态的属性变量不会被GC清除)，除非整个JVM退出了。这个问题我之前也想几天，并 且自己写代码来做了个实验。
>
>观点三：（Good！ ）
由于DAO的初始化，会比较占系统资源的，如果用静态方法来取，会不断地初始化和释放，所以我个人认为如果不存在比较复杂的事务管理，用 singleton会比较好。个人意见，欢迎各位高手指正。  
>参考链接:
>[静态类和单例模式区别](https://www.cnblogs.com/zhtao_tony/p/3956047.html)

## 2. 管理bean的生命周期
对Spring调用bean构造方法的探究(告诉Spring 如何构造对象)
### 2.1 重写构造方法加入打印调用，发现的确有输出
### 2.2 将构造器声明为private也能被创建
[反射破坏单例的私有构造函数保护](https://blog.csdn.net/tiwerbao/article/details/20838903)  
帮助理解Spring为何能做到这点
### 2.3. 三种构造方法(告诉Spring怎样构建对象)
#### 2.3.1. 直接调用Constructor(Instantiation with a Constructor)
```xml
<bean id="testDAO" class="testspringcore.TestDAO">
</bean>
```
最普通的方法，默认新建单例模式的对象，标签内增加`scope="prototype"`可以允许新建多个实例。
Spring会在最初的`ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");`
步骤自动创建所有被注册的单例模式的Java bean，标签内增加`lazy-init="true"`则指定在需要时在创建。  

#### 2.3.2. 绑定getInstance() (Instantiation with a Static Factory Method)
```xml
<bean id="testBean" class="testspringcore.TestBean" factory-method="getInstance" scope="prototype">
</bean>
```
这种方法的好处是可以手动规定创建实例的方法，例如通过绑定如下getInstance方法可以规定该类最多被创建10次。
```java
//only have ten instance
//This class has the function of producing instance

public class TestBean {

    private static int count = 0;

    public static TestBean getInstance()
    {
        System.out.println("factory method invoked");
        if(count<10)
        {
            count++;
            return new TestBean();
        }
        return null;
    }
}
```
#### 2.3.3. 绑定Factory.class (Instantiation by Using an Instance Factory Method)
```xml
<!-- register factory bean -->
<bean id="testBean2Factory" class="testspringcore.TestBean2Factory"></bean>

<!-- register testbean2 -->
<bean id="testBean2" factory-bean="testBean2Factory" factory-method="getInstance" scope="prototype"></bean>
```
与方法2类似，不过这时我们为要新建的bean类构造了一个对应的Factory类。
## 3. 管理bean的依赖（依赖注入 Dependency Injection）
可以理解为「管理bean的生命周期」是创建一个实例，「管理bean的依赖」是对这个实例的属性值进行初始化。
### 3.1. 使用XML方式实现依赖注入(2种方式)
spring会在创建@bean testService之前先创建一个它要依赖的bean的实例
#### 3.1.1. setter注入（常用）
*applicationContext.xml*
```xml
<bean id="testDAO" class="testspringcore.TestDAO" lazy-init="true">
</bean>

<bean id="testService" class="testspringcore.TestService">
    <property name="testDAO" ref="testDAO"></property>
</bean>
```
多个属性即多行
```xml
<bean id="testDAO" class="testspringcore.TestDAO" lazy-init="true">
</bean>
<bean id="userDAO" class="testspringcore.UserDAO">
</bean>

<bean id="testService" class="testspringcore.TestService">
    <property name="testDAO" ref="testDAO"></property>
    <property name="userDAO" ref="userDAO"></property>
</bean>
```
`<property>`标签内的name对应的是testService的属性，ref对应的是该xml文件其他bean的id  
TestService类需要有各个属性的setter  
*TestService.java*
```java
private TestDAO testDAO;
private UserDAO userDAO;

public void setTestDAO(TestDAO testDAO) {
    this.testDAO = testDAO;

}

public void setUserDAO(UserDAO userDAO) {
    this.userDAO = userDAO;
}
```
此处的setter一定要严格遵守setter方法名命名规范「set+Property驼峰式」，可以用IDE自带的代码生成工具生成setter。
#### 3.1.2. 构造注入
（1）普通构造注入  
*applicationContext.xml*
```xml
<bean id="testDAO" class="testspringcore.TestDAO" lazy-init="true">
</bean>
<bean id="userDAO" class="testspringcore.UserDAO">
</bean>

<bean id="testService" class="testspringcore.TestService">
    <constructor-arg ref="testDAO"></constructor-arg>
    <constructor-arg ref="userDAO"></constructor-arg>
</bean>
```
constructor-arg的顺序与构造器参数顺序不用保持一致，可以自动识别，但是如果有重复类型的属性或者有原始类型的属性，需要注明对应关系。  
*TestService.java*
```java
private TestDAO testDAO;
private UserDAO userDAO;

public TestService(UserDAO userDAO, TestDAO testDAO) {
    this.userDAO = userDAO;
    this.testDAO = testDAO;
}
```
***
（2）有简单类型的构造注入
有简单数据类型的构造注入：
> 简单数据类型 = 原始类型 + String（String虽然是reference类型但是有很多原始类型的特征）  

*applicationContext.xml*
```xml
<bean id="exampleBean" class="testspringcore.ExampleBean">
    <constructor-arg value="7500000"></constructor-arg>
    <constructor-arg value="42"></constructor-arg>
</bean>
```
- value内必须用引号，体现不出数据类型，可以用`type`属性注明
  ```xml
  <bean id="exampleBean" class="testspringcore.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
  </bean>
  ```
- 可以使用`index`属性注明传参数据增强可读性
  ```xml
  <bean id="exampleBean" class="testspringcore.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
  </bean>
  ```
- 利用`name`属性注明构造器参数名
  ```xml
  <bean id="exampleBean" class="testspringcore.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
  </bean>
  ```
- setter注入也是可以注入简单类型的，不多做介绍
- 什么时候简单类型需要被注入？  
  如果我们的代码只被自己使用，其实可以直接在类的属性里给简单数据类型初始化赋值，但是例如如果我们要使用jar包分享我们的代码给其他人使用，就应该利用xml文件进行依赖注入，因为这样不需要重新编译代码，jar包内是二进制文件不是java源代码是不能被修改的。
#### 3.1.3. setter注入 vs 构造器注入
- spring官方文档推荐构造器注入
- 费老师实际工作常用setter注入
- 具体使用哪一种方法应该由团队成员讨论后由leader决定

#### 3.1.4. collection作为参数
collection包括：
- list
- set
- map

*ExampleBean.java*
```java
private List<String> list;
private Set<String> set;
private Map<String, String> map;
```
增加三个集合属性  
*applicationContext.xml*  
```xml
<bean id="exampleBean" class="testspringcore.ExampleBean" p:years="7500000" p:ultimateAnswer="42">
    <property name="list">
        <list>
        <value>str1</value>
        <value>str2</value>
        <value>str3</value>
        </list>
    </property>
    <property name="set">
        <set>
        <value>strset1</value>
        <value>strset2</value>
        <value>strset3</value>
        </set>
    </property>
    <property name="map">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value="myDataSource"/>
        </map>
    </property>
</bean>
```
`p:preperty`是另一种更简单的方式用来注入简单数据类型  
>我们一般不常用集合，但是有时候引用的jar包需要我们以collection的方式注入。

#### 3.1.5. properties读取
类似集合，常用类似方法读取properties配置  
*applicationContext.xml*  
```xml
<property name="adminEmails">
    <props>
        <prop key="administrator">administrator@example.org</prop>
        <prop key="support">support@example.org</prop>
        <prop key="development">development@example.org</prop>
    </props>
</property>
```
#### 3.1.6. depends-on
之前我们用ref来指明依赖的类，但有时这种依赖关系不是很明显，这要求我们用`depends-on`属性强制指明依赖的类。  
*applicationContext.xml*  
```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```
`depends-on`内可用分隔符分隔指明多个依赖关系

#### 3.1.7. Lazy-initialized beans
前面我们已经提到过，这代表延迟加载单例bean  
以前我们不这样做，但是现在内存等硬件条件上升，我们常常对单例实例不使用延迟加载，我们用空间换时间，为了更好性能。  
*程序使用`ClassPathXmlApplicationContext()`注入时只默认注入全部的单例类，非单例类我们不知道该创建多少个，也没必要提前创建*

### 3.2. 使用注解方式实现依赖注入
- @Component->通用组件  
  泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。
- @Controller->控制器  
  用于标注控制层组件（如struts中的action）
- @Service-> Service  
  用于标注业务层组件
- @Repository -> DAO  
  用于标注数据访问组件，即DAO组件
*@Component泛指所有组件，其他三个是拥有特殊语义的注解*
*Spring鼓励使用其他三个更具有语义化的注解，会在日后支持提供feature*

#### 3.2.1. demo
*applicationContext.xml*
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>
    
    <!-- specify where beans locate -->
    <context:component-scan base-package="testspringannotation"></context:component-scan>
</beans>
```
>此时的xml配置文件很简洁，指明了我们用注解的方式来注入依赖。  
*TestController.java*
```java
@Component
public class TestController {
	
	//we are sure, Spring uses Java Reflection mechanism to update its private field
	
	@Autowired
	private TestService testService;
	
	public void outputcollaborators()
	{
		//invoke method the service
		testService.outputinfo();
	}
}
```
#### 3.2.2. @Autowired
>wire, 装载；装配  
>autowired, 自动装载  

我们用`@Autowired`注明哪些类需要被注入，方式有很多种：
1. 在构造器上方注明
   ```java
    public class MovieRecommender {
        private final CustomerPreferenceDao customerPreferenceDao;

        @Autowired
        public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
            this.customerPreferenceDao = customerPreferenceDao;
        }
    }
    ```
2. 在setter函数上方注明
    ```java
    public class SimpleMovieLister {
        private MovieFinder movieFinder;

        @Autowired
        public void setMovieFinder(MovieFinder movieFinder) {
            this.movieFinder = movieFinder;
        }
    }
    ```
3. 在具有任意名称和多个参数的方法上方注明
   ```java
    public class MovieRecommender {
        private MovieCatalog movieCatalog;
        private CustomerPreferenceDao customerPreferenceDao;

        @Autowired
        public void prepare(MovieCatalog movieCatalog,
                CustomerPreferenceDao customerPreferenceDao) {
            this.movieCatalog = movieCatalog;
            this.customerPreferenceDao = customerPreferenceDao;
        }
    }
4. 在成员变量fields上注明，甚至可以混用注解方法
    ```java
    public class MovieRecommender {
        private final CustomerPreferenceDao customerPreferenceDao;

        @Autowired
        private MovieCatalog movieCatalog;

        @Autowired
        public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
           this.customerPreferenceDao = customerPreferenceDao;
        }
    }
    ```
    显然，在成员变量上使用`@Autowired`用到了Java的反射机制。这种方法有一套机制去唯一标识注入的对象，通常先byType，若有冲突再转而byName。
### 3.3. 思考
#### 3.3.1. 框架设计原则：约定优于配置  
   例如约定bean默认不延迟加载。事无巨细的配置降低了框架的便利性。
#### 3.3.2. MVC  
   <img src="https://ws3.sinaimg.cn/large/006tNc79ly1g2w91nprxrj31000mewkd.jpg" width="500rpx"/>   

MVC是所有web项目为了实现“高内聚，低耦合”应遵循的模式。  

#### 3.3.3. 对比基于XML与基于注解的注入方式
- 显然有各自的优缺点
- 不能因为基于注解的方式是较新的就认为它更好
- [参考资料:spring注解和xml配置的优缺点比较](http://blog.csdn.net/y6s6y6y/article/details/80992833)
- 官方文档：
  - 注解更贴近代码，有更多context，配置更短更精确；
  - 但是违反了POJO的模式，配置变得分散更难控制；
  - Spring支持两种方式混用，结合各自的有点做出更好的配置；
  - 值得指出的是，通过其JavaConfig选项，Spring允许以非侵入方式使用注释，而无需触及目标组件源代码，并且在工具方面，Spring Tool Suite支持所有配置样式。

#### 3.3.4. 再议依赖注入
[用小说的形式讲解为什么需要依赖注入](https://zhuanlan.zhihu.com/p/29426019)
>Spring框架的核心功能之一就是通过依赖注入的方式来管理Bean之间的依赖关系。那么我们今天的主角依赖注入到底有什么神奇之处呢？请往下继续看。  
>了解过设计模式的朋友肯定知道工厂模式吧，即所有的对象的创建都交给工厂来完成，是一个典型的面向接口编程。这比直接用new直接创建对象更合理，因为直接用new创建对象，会导致调用者与被调用者的硬编码耦合；而工厂模式，则把责任转向了工厂，形成调用者与被调用者的接口的耦合，这样就避免了类层次的硬编码耦合。这样的工厂模式确实比传统创建对象好很多。但是，正如之前所说的，工厂模式只是把责任推给了工厂，造成了调用者与被调用者工厂的耦合。  
>Spring框架则避免了调用者与工厂之间的耦合，通过spring容器“宏观调控”，调用者只要被动接受spring容器为调用者的成员变量赋值即可，而不需要主动获取被依赖对象。这种被动获取的方式就叫做依赖注入，又叫控制反转。依赖注入又分为设值注入和构造注入。而spring框架则负责通过配置xml文件来实现依赖注入。而设值注入和构造注入则通过配置上的差异来区分。

**依赖注入 == 控制反转**

# Day 6 - Day X: Spring AOP
## 1. 了解AOP

### 1.1. 名词解释
- AOP: Aspect-Oriented Programming: 面向切面编程  
  分布于应用中多处的功能称为横切关注点，通过这些横切关注点在概念上是与应用的业务逻辑相分离的，但其代码往往直接嵌入在应用的业务逻辑之中。将这些横切关注点与业务逻辑相分离正是面向切面编程（AOP）所要解决的。切面实现了横切关注点的模块化  
  一句话概括：切面是跟具体业务无关的一类共同功能。
- advice: 通知
- pointcut: 切入点
- weaving: 织入

## 2. 配置
这里我们不用xml用注解配置，例如`@Before("execution(* com.neuedu.model.service.AccountService.*(..))")`
### 2.1. @Before
### 2.2. @AfterReturning
### 2.3. @AfterThrowing
业务方法一定要抛出异常（try catch throw / throws）皆可，否则`@AfterThrowing`注解无法起作用。
### 2.4. @After
### 2.5. 执行顺序探究
```java
try{
    try{
        //@Before
        method.invoke(..);
    }finally{
        //@After
    }
    //@AfterReturning
}catch(){
    //@AfterThrowing
}
```
`@After`和`@AfterReturning`注解的执行顺序以及生成的动态代理类结果在新版本中有变化，有待进一步探究。
[aop after-returning 和after的区别？](http://www.imooc.com/qadetail/75298)

## 3.Demo
JDBC事务处理是经常使用AOP的一个典型事例（其他还有诸如日志等），我们举例对比使用AOP前后代码量与代码结构的变化。
### 3.1. 使用切面之前
*DBUtils.java*
```java
public class DBUtils {
	
	private static ThreadLocal<Connection> tl = new ThreadLocal<>();
	
	static
	{
		//加载数据库驱动
		try {
			Class.forName("com.mysql.jdbc.Driver");
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
	public static void beginTransaction()
	{
		//1.得到数据库连接
		Connection conn = getConnection();
		//2.设置自动提交为false    
		try {
			conn.setAutoCommit(false);
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}		
	}
	
	public static Connection getConnection()
	{
		Connection conn = tl.get();
		if(conn==null)
		{			
			try {
				conn = DriverManager.
						getConnection("jdbc:mysql://localhost:3306/mydb?useUnicode=true&characterEncoding=utf8", "root", "root");
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			//放在本地线程
			tl.set(conn);
		}
		
		return conn;
	}
	
	public static void commit()
	{
		//1.得到数据库连接
		Connection conn = getConnection();
		//2. 提交
		try {
			conn.commit();
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
	public static void rollback()
	{
		//1.得到数据库连接
		Connection conn = getConnection();
		//2. 提交
		try {
			conn.rollback();
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
	public static void close()
	{
		//1.得到数据库连接
		Connection conn = getConnection();
		//2. 提交
		try {
			conn.close();
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		//3.把conn从tl中移除
		tl.remove();
	}
}
```
使用`ThreadLocal`：保证连接是共享的，同一个连接

*Service类 Service.java*
```java
@Service
public class AccountService {
	
	@Autowired
	AccountDAO accountDAO;
	
	public void transferMoney()
	{		
		//1. 获得数据库连接，开启事务
		DBUtils.beginTransaction();
		try
		{			
			accountDAO.deduct();
			accountDAO.add();
			
			DBUtils.commit();
		}
		catch(Exception e)
		{
			DBUtils.rollback();
			e.printStackTrace();
		}
		finally
		{
			DBUtils.close();
		}
	}
	
	public static void main(String[] args) {
		//1. 开启spring容器
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
		
		AccountService accountService = (AccountService)context.getBean("accountService");
		
		accountService.transferMoney();
	}
}
```
*AccountDAO.java*
```java
@Repository
public class AccountDAO {
	
	public void deduct() throws SQLException
	{
		//1. 获得连接
		Connection conn = DBUtils.getConnection();
		
		PreparedStatement ps = conn.prepareStatement("update account set balance = balance-10 where accountid =2");
	    ps.executeUpdate();	     
	}
	
	public void add() throws SQLException
	{
		//1. 获得连接
		Connection conn = DBUtils.getConnection();
		
		PreparedStatement ps = conn.prepareStatement("update account set balance = balance+10 where accountid =1");
	    ps.executeUpdate();
	}
}
```
### 3.2. 使用切面之后
*切面类 TransactionAspect.java*
```java
@Component
@Aspect
public class TransactionAspect {
	
	@Before("execution(* com.neuedu.model.service.AccountService.*(..))")
	public void getConnection()
	{
		DBUtils.getConnection();
	}
	
	@AfterReturning("execution(* com.neuedu.model.service.AccountService.*(..))")
	public void commitConnection()
	{
		DBUtils.commitConnection();
		DBUtils.closeConnection();
		
	}
	
	@AfterThrowing("execution(* com.neuedu.model.service.AccountService.*(..))")
	public void rollbackConnection()
	{
		DBUtils.rollbackConnection();
		DBUtils.closeConnection();
	}
	
	@After("execution(* com.neuedu.model.service.AccountService.*(..))")
	public void closeConnection()
	{
		DBUtils.closeConnection();		
	}
}
```
*Service类 MyService.java*
```java
class MyService
{
      public void test()
      {       
           MyDAO myDAO = new MyDAO();
           myDAO.deduct();          
           myDAO.add();         
       }
}
```
可见，Service类被简化了，目前Service只有一个方法，显然可以预测随着业务的增多Service将提供越来越多的业务函数，利用切面将只关注真正的业务逻辑而不再需要写context比较低的代码。

## 3. 探究AOP实现机制：动态代理  
**注意:** AOP不是代码拷贝，是方法调用  
有两种动态代理方式
### 3.1. 运行时创建临时对象
- 方法1: 实现一个接口，spring直接调用jdk，创建一个类，实现一个接口
- 方法2: 没实现接口，spring调用内置的cglib

### 3.2. 编译过程修改编译生成的class文件
- Aspectj修改.class文件
- Spring AOP依赖IOC

## 4. @Around
利用`@Around`我们可以显式地指明语句插入的位置，利用`@Around`代替上面的四个注解完成JDBC事务管理如下：
*TranscationAspect.java*
```java
	@Around("execution(* com.neuedu.model.service.AccountService.*(..))")
	public void process(ProceedingJoinPoint pjp) throws Throwable {
		DBUtils.getConnection();
		try {
			pjp.proceed();

			DBUtils.commitConnection();
		} catch (Throwable e) {
			System.out.println(e);

			DBUtils.rollbackConnection();

			throw e;
		} finally {
			DBUtils.closeConnection();
		}
	}
```
我们可以看到，这和前四个注解共同使用的实现逻辑类似，其中传入参数`ProceedingJoinPoint pjp`代表切入点
```java
try{
    try{
        //@Before
        method.invoke(..);
    }finally{
        //@After
    }
    //@AfterReturning
}catch(){
    //@AfterThrowing
}
```
只不过利用`@Around`注解我们手动实现了其中的 try, finally, catch关系，这有助于我们更好地理解AOP切入的位置。

## 5. XML配置
下面再来尝试不用注解用XML配置切面  
使用场景
- 用了jar包里的Aspect类
- xml配置便于集中管理

```java
@Component
public class LogAspect {
	
	public void before()
	{
		//before the method executes:
		System.out.println("methods before");
	}
	
	public void after()
	{
		//after the method executes:
		System.out.println("methods after");
	}
	
	public void afterthrowing()
	{
		System.out.println("methods exception");
	}

	public void afterreturnning()
	{
		System.out.println("methods runs without exception");
	}
}
```

## 6. Advisor顾问
Advisor顾问：只有一个方法的Aspect切面  
（因为`@Around`很流行，使用很多切面类里只有一个方法，如果一个切面只有一个advice，我们叫它advisor。）
### 6.1. 配置方法
```xml
<aop:config>
    <aop:advisor advice-ref="transactionAdvisor" pointcut="execution(* com.neuedu.model.service.AccountService.*(..))"/>
</aop:config>
```
```java
@Component
public class TransactionAdvisor implements MethodInterceptor {
	/**
	 * because a advisor only have one advice, to specify the type of the advice, 
	 * you must implements different interface.
	 * 
	 * because we must implement a interface, this is called invasive design（侵入式设计） 
	 * which is actually not good.
	 */

	@Override
	public Object invoke(MethodInvocation arg0) throws Throwable {
		
		DBUtils.getConnection();
		try
		{
			//call our business logic
			//dao.deductMoney();
			//dao.addMoney();
			arg0.proceed();
			
			DBUtils.commitConnection();
		}
		catch (Throwable e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			
			DBUtils.rollbackConnection();
			
			throw e;
		}
		finally
		{
			DBUtils.closeConnection();
		}
		
		return null;
	}
}

```
通过实现接口代表是哪一种切面
### 6.2. 优点
- 配置简化
- 便于使用第三方切面

### 6.3. 小结
这体现了框架是在不断演变的，框架设计者也在不断根据用户使用情况及反馈调整框架的设计。如此例，设计者发现很多只有一个方法的切面类也需要繁琐的
配置因而引入了顾问的概念帮助简化这些只有一个方法的切面的配置。  
各种主流框架一般都支持xml配置和注解配置，xml配置和注解配置是两套并行的配置方法。Java在5.0版本中引入了注解，之后注解迅速流行，但是对于
一些早期项目，考虑到项目稳定性以及团队成员的学习问题还更喜欢xml，当然xml配置也有自身的有点，在上面提到过最显著的两点。

## 7. MyBatis + Spring
**核心问题：**MyBatis实现IOC由Spring创建并控制
>数据库连接池

### 7.1. 分析配置文件
由配置文件即可看出配置的思想：配置SqlSession Fac
*applicationContext.xml*
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd">

	<!--注明通过注解来配置AOP-->
    <context:annotation-config/>
    <context:component-scan base-package="com.neuedu.model.service"></context:component-scan>
    
    <!-- to enable AOP aspect weave -->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
    
    <!-- connect spring with mybatis -->
    <!-- 配置数据源，类似于属性文件，下面会用到 -->
    <bean id="ds" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://localhost:3306/scott"/>
		<property name="username" value="root" />
		<property name="password" value="root" />
	</bean>
	
    <!-- (1) -->
	<!-- configure MyBatis sessionFactory -->
	<!-- 配置MyBatis的sessionFactory，将其交由Spring管理 -->
	<bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<!-- 加载mybatis的主配置文件 -->
		<property name="configLocation" value="classpath:mybatis-config.xml"/>
		<!-- 注入数据源 -->
		<property name="dataSource" ref="ds" />
	</bean>
	
    <!-- (2) -->
	<!-- tell Spring about Mapper information -->
	<!-- MyBatis: 指定mapper，即可用的DAO方法 -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<!-- 指定扫描的包名，如果扫描多个包，每个包中间使用半角逗号分隔-->
		<property name="basePackage" value="com.neuedu.model.mapper"/>
		<!-- 自动创建session（connection）去数据库交互 -->
		<property name="sqlSessionFactoryBeanName" value="sessionFactory"/>
	</bean>
	
    <!-- (3) -->
    <!-- 以下为Spring AOP的配置 -->
	<!-- configure Spring transaction manager which is a advisor -->
	<!-- 配置AOP，该事务AOP由Spring提供 -->
	<bean id="txm" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="ds" />
	</bean>
	
	<!-- if we want to use annotation to control transaction -->
	<!-- 指定利用注解的方式配置 -->
	<tx:annotation-driven transaction-manager="txm"/>

</beans>
```
分析利用Spring来管理MyBatis的配置文件需要回想之前手动管理MyBatis的步骤
- (1) 配置MyBatis的sessionFactory
之前我们采用如下方式新建sessionFactory
```java
String resource = "mybatis-config.xml";
InputStream inputStream;
SqlSession session = null;
try {
    // 1. Create a sqlsessionFactory
    inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
}
```
- (2) 指定可以使用的mapper实现
- (3) 配置Spring AOP
**这其中，之前由我们手写的例如TransactionManager切面类等都由Spring提供的类来实现！配置一次后其他的使用就很简单了！**  
**初次接触配置文件可能比较陌生，想要理解好要回想不用Spring框架我们要做哪些工作**

## 8. 其他
### 8.1. 补充知识点
- 判断是否是同一个对象：打印对象地址。

- 什么时候把类加载到内存？三种情况：
  - new类的对象
  - 调用类的静态方法
  - `Class.forName("");`

- JDBC事务
  >事务是必须满足4个条件（ACID）
  >事务的原子性（ Atomicity）：一组事务，要么成功；要么撤回。  
    一致性 （Consistency）：事务执行后，数据库状态与其他业务规则保持一致。如转账业务，无论事务执行成功否，参与转账的两个账号余额之和应该是不变的。  
    隔离性（Isolation）：事务独立运行。一个事务处理后的结果，影响了其他事务，那么其他事务会撤回。事务的100%隔离，需要牺牲速度。  
    持久性（Durability）：软、硬件崩溃后，InnoDB数据表驱动会利用日志文件重构修改。可靠性和高速度不可兼得， innodb_flush_log_at_trx_commit 选项 决定什么时候吧事务保存到日志里。

- DML
  SQL分为DML数据库操纵语言（SELECT INSERT DELETE UPDATE）和DDL数据库定义语言（创建表时的一些定义操作ALTER等）  
  JDBC默认自动提交，即`executeUpdate()`后自动执行了commit()。但是我们想保持数据库的原子性，所以在上面通过`conn.setAutoCommit(false);`禁用自动提交。禁用后支持`rollback()`回滚操作及`rollback(Savepoint savepoint) `回滚到固定位置。数据库在`commit()`前都没被真正改变，都能通过`rollback()`撤销之前做的`executeUpdate()`操作。数据库应该只是有一个临时的镜像而已。COMMIT命令用于把事务所做的修改保存到数据库。

### 8.2. 再议错误处理
- try catch
  自行处理
- throws
  交给上级处理
- try catch + throw
  自行处理+通知上级发生过异常

不能在主方法里向上抛出异常

### 8.3. Context
努力实现单位代码量包含更多Context