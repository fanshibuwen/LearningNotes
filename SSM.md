==源码无法下载==
$$
mvn dependency:resolve -Dclassifier=sources
$$

# Spring

## 常用依赖

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>14</source>
                    <target>14</target>
                </configuration>
            </plugin>
        </plugins>
        </build>
```
## 常用配置

```xml
<!--    注解支持-->
    <context:annotation-config/>
<!--    扫描指定的包-->
    <context:component-scan base-package="xxx"/>
<!--    让SpringMVC不处理静态资源-->
    <mvc:default-servlet-handler/>
<!--    支持注解驱动 (代替 配置处理器映射器和处理器适配器 的工作)-->
    <mvc:annotation-driven/>
```
## 注解说明

- @Autowire：自动装配。先通过类型，如果一个类型有多个bean则使用id进行匹配，
还可以和@Qualifier(value = "xxx")//Qualifier的意思是，
当IOC容器中有多个cat时，指定这个cat为指定id的cat
- @Nullable：字段标记了这个注解，则说明这个字段可以为null
- @Resource：自动装配通过名字。类型


- @Component：组件。放在类上，说明此类已经被Spring所管理，就是bean。相当于
```xml
<bean id="dog" class="com.pojo.Dog"/>
```
- @Value：注入值，相当于bean里的properties里的value
- @Repository：标注此类为dao层。但作用和@Component一样
- @Service：标注此类为service层。但作用和@Component一样
- @Controller：标注此类为web层。但作用和@Component一样
- @Scope:设置此类为单例或者原型
- @Configuration：标注此类为配置类，就相当于之前的beans.xml
- @Bean：将返回值做为一个bean放在Spring容器中
- @RequestMapping("/add"):请求路径为/add,加在类上为父路径,加在方法上为子路径
- @PathVariable:用在方法参数前,意为将此参数映射到子路径中,例如,@RequestMapping("/add/{a}/{b}")
- @GetMapping
- @DeleteMapping
- @PostMapping
- @PutMapping:以上四种为通过不同的请求方式来请求
- @ResponseBody:只要该方法加了这个注解,就不会走视图解析器,会直接返回一个字符串,实现前后端分离
- @RestController:放在类上,如果加了这个注解,那么这个类中的所有方法都只会返回json字符串

## AOP

### 动态生成代理对象万能模板

- 接口

```java
package com.demo02;
public interface UserService {
    public void add();
    public void delete();
    public void update();
    public void query();
}
```

- 实现类

```java
public class UserServiceImpl implements UserService{
    @Override
    public void add() {
        System.out.println("增加了一个用户");
    }
    @Override
    public void delete() {
        System.out.println("删除了一个用户");
    }
    @Override
    public void update() {
        System.out.println("修改了一个用户");
    }
    @Override
    public void query() {
        System.out.println("查询了一个用户");
    }
}
```

### 自动生成代理类

```java
public class ProxyInvocationHandler implements InvocationHandler {
    //被代理的接口
    private Object target;
    public void setTarget(Object target) {
        this.target = target;
    }
    //生成得到的代理类
    public Object getProxy() {
        return Proxy.newProxyInstance(this.getClass().getClassLoader(),
                target.getClass().getInterfaces(), this);
    }
    //处理代理实例，并返回结果
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log(method.getName());
        //调用代理对象的任何方法  实质执行的都是invoke（）方法
        Object result = method.invoke(target, args);
        return result;
    }
    public void log(String msg) {
        System.out.println("执行了" + msg + "方法");
    }
}
```

- main

```java
public class Client {
    public static void main(String[] args) {
       //真实角色
        UserServiceImpl userService = new UserServiceImpl();
        //生成角色，不存在
        ProxyInvocationHandler pih = new ProxyInvocationHandler();
        //设置代理对象
        pih.setTarget(userService);
        //动态生成代理类
        UserService proxy = (UserService) pih.getProxy();
        proxy.delete();
    }
}
```

### 使用Spring实现AOP

- 目标对象

```java
public class UserServiceImpl implements UserService{
    @Override
    public void add() {
        System.out.println("增加了一个用户");
    }
    @Override
    public void delete() {
        System.out.println("删除了一个用户");
    }
    @Override
    public void update() {
        System.out.println("修改了一个用户");
    }
    @Override
    public void query() {
        System.out.println("查询了一个用户");
    }
}
```

### 使用Spring的原生接口[主要是Spring]

- 前置

```java
public class Log implements MethodBeforeAdvice {
    //method：要执行的目标对象的方法
    //args：参数
    //target：目标对象
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println(target.getClass().getName()+"的"+method.getName()+"被执行了");
    }
}
```

- 后置

```java
public class AfterLog implements AfterReturningAdvice {
    //returnValue：返回值
    @Override
    public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
        System.out.println("执行了"+method.getName()+"方法，返回了"+returnValue);
    }
}
```

- 配置

```xml
<!--    配置aop-->
    <aop:config>
<!--        切入点-->
        <aop:pointcut id="pointcut" expression="execution(* com.service.UserServiceImpl.*(..))"/>
<!--        执行增强-->
        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
    </aop:config>
```

### 自定义实现AOP[主要是切面定义]

- 切面

```java
public class demo {
    public void beforeMethod(){
        System.out.println("前置增强");
    }
    public void afterMethod(){
        System.out.println("后置增强");
    }
}
```

- 配置

```xml
<!--    自定义类-->
    <bean id="method" class="com.diy.demo"/>
    <aop:config> 
<!--        自定义切面-->
        <aop:aspect id="point" ref="method">
<!--            切点-->
            <aop:pointcut id="point" expression="execution(* com.service.UserServiceImpl.*(..))"/>
<!--            增强-->
            <aop:before method="beforeMethod" pointcut-ref="point"/>
            <aop:after method="afterMethod" pointcut-ref="point"/>
        </aop:aspect>
    </aop:config>
```

### 使用注解

```java
@Aspect
@Component
public class AnnotationPointCut {
    @Before("execution(* com.service.UserServiceImpl.*(..))")
    public void before(){
        System.out.println("前置=========");
    }
}
```

```xml
<!--    开启注解支持  这里不能使用<context:annotation-config/>-->
    <aop:aspectj-autoproxy/>
    <!--    扫描指定的包-->
    <context:component-scan base-package="com.diy"/>
```

### Bean的生命周期：

- 创建前准备：Bean在开始加载之前要从上下文和一些配置中去解析并查找Bean有关的扩展实现。比如init-method，容器在初始化Bean的时候要调用的方法，destory-method，容器在销毁Bean的时候要调用的方法。这些类和配置其实是Spring提供给开发者用来去实现Bean加载过程中的一些扩展。
- 创建实例：通过反射去创建Bean的实例化对象，并且会扫描和解析Bean声明的一些属性
- 依赖注入阶段：如果被实例化的Bean存在依赖其他Bean对象的一些情况，则需要对这些依赖的Bean进行对象注入
- 容器缓存阶段：把Bean保存到容器已经Spring的缓存中，到了这个阶段Bean就可以被开发者进行使用了，比如init-method会在这个阶段被调用
- 销毁实例阶段：当Spring的应用上下文被关闭的时候，那么这个Spring上下文中所有的Bean会被销毁，destory-method方法会在这个阶段被调用

#  SpringMVC

## 处理流程(原理 在实际开发中不会这么用)

![SpringMVC执行流程](C:\Users\zhao\Desktop\SpringMVC执行流程.png)

==刚开始会找web.xml文件,执行以下代码(做一个SpringMVC项目的时候这个配置是死的,每次都需要配置)==

```xml
<!--    配置DispatcherServlet:这个是SpringMVC的核心;请求分发器,前端控制器-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
<!--        DispatcherServlet要绑定Spring的配置文件-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
<!--        启动级别:1代表和服务器一起启动-->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

<!--    配置SpringMVC的乱码过滤问题-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

上述代码执行后,映射到spring-mvc.xml中

```xml
<!--    配置处理器映射器-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
<!--    配置处理器适配器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
<!--    配置视图器解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
<!--BeanNameUrlHandlerMapping:bean-->
    <bean id="/hello" class="controller.HelloController"/>
```

- 处理器映射器映射到/hello,将处理器执行链返回
- 处理器适配器定位到controller.HelloController内执行相应的业务逻辑
- 将返回的ModelAndView到视图解析器,进行相应的拼接后进行相应的渲染,展示在页面上
  ##创建SpringMVC项目时首先要做的工作,在spring-mvc.xml中加入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation=
       "http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
">
<!--    扫描指定的包-->
    <context:component-scan base-package="xxx"/>
<!--    让SpringMVC不处理静态资源-->
    <mvc:default-servlet-handler/>
<!--    支持注解驱动 (代替 配置处理器映射器和处理器适配器 的工作)-->
    <mvc:annotation-driven/>
    <!--    配置视图器解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

## 解决json乱码问题的spring-mvc.xml

```xml
<!--    解决json字符串乱码问题的配置-->
    <mvc:annotation-driven>
        <mvc:message-converters register-defaults="true">
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8"/>
            </bean>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="objectMapper">
                    <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                        <property name="failOnEmptyBeans" value="false"/>
                    </bean>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```

# Mybatis

## 编写一个工具类

==此工具类的目的就是获取SqlSession==

```java
//sqlSessionFactory
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            //使用Mybatis第一步：获取sqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    //既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例。
    // SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```

## 可能遇到的问题

==maven由于约定大于配置，我们之后可能遇到我们写的配置文件，无法被导出或者生效的问题，解决方案：==

```xml
<!--    在build中配置resources，来防止我们的资源导出失败的问题-->
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```

## 做mybatis项目一般流程

- 编写工具类

- 编写mybatis-config.xml（里面写核心配置文件）

  - ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
      <environments default="development">
        <environment id="development">
          <transactionManager type="JDBC"/>
          <dataSource type="POOLED">
            <property name="driver" value="${driver}"/>
            <property name="url" value="${url}"/>
            <property name="username" value="${username}"/>
            <property name="password" value="${password}"/>
          </dataSource>
        </environment>
      </environments>
      <mappers>
        <mapper resource="Mapper.xml的全路径名"/>
      </mappers>
    </configuration>
    ```

- 写pojo

- 写dao里的接口（这里多数命名为Mapper）

- 写dao里接口的对应xml（这个xml就相当于是impl实现类）

  - ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!--namespace=绑定一个Dao/Mapper接口-->
    <mapper namespace="com.dao.UserMapper">
        <select id="getUserList" resultType="com.pojo.User">-- 这里的id就相当于实现接口
            select * from mybatis.user where id = #{id};
        </select>
    </mapper>
    ```
    
  - 这里的namespace就是要编写CRUD的接口

  - id就是接口里对应的方法

  - resultType就是返回的就是返回的结果类型（这里要写全包名）

  - parameterType就是传递的参数的类型

  - #{id}就是pojo里的属性

- 将刚编写的xml注册到mybatis-config.xml中

  - ```xml
    <!--    每一个Mapper.xml都需要在mybatis核心配置文件中注册-->
        <mappers>
            <mapper resource="com/dao/UserMapper.xml"/>
        </mappers>
    ```

- 编写测试类

  - 注意：这里的测试类里执行增删改操作，需要提交事务之后才能生效

    - ```java
      sqlSession.commit();
      ```

  - 要提交事务，那么我么就要在建表的时候在结尾加上engine=innodb语句将数据库的存储引擎改为innodb

## 万能的map

​		当pojo里的字段比较多的时候，我们这时候如果使用pojo作为parameterType，那么在Test中，由于传递参数是pojo的类型，那么此时我们只能通过new对象的方式来传递参数，此时就需要写下pojo的所有参数才能通过有参构造传递过去。

​		那么此时Map<String,Object>就可以很好的解决此类麻烦，我们只需要插入自己所需要的字段就可以完成插入操作。

```java
@Test
    public void test4(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String, Object> map = new HashMap<String, Object>();

        map.put("userId",1);
        
        User userById2 = mapper.getUserById2(map);
        System.out.println(userById2);
    }
```

- map传参：在sql中取出key即可
- 对象传参：在sql中取出对象的属性即可
- 只有一个基本类型的参数时，可以直接在sql中取到

## 配置解析

### 环境配置（environments）

Mybatis可以配置成适应多种环境

但是尽管可以配置多个环境，但每个SqlSessionFactory实例只能通过default来选择一种环境使用

Mybatis的默认事务管理器就是JDBC，连接池就是POOLED

### 属性（properties）

- 在mybatis-config.xml核心配置文件的前边写入
- 里面是resource，所以直接定位的就是资源文件夹，不可以再加classpath
- 可以在其中增加一些属性配置
- 如果标签体内的字段和引入的资源文件里的字段名有相同的，则优先使用配置文件里的

### 类型别名（typeAliases）

- 用一个较为简短的名字来代替之前返回的结果类型中  resultType="com.pojo.User"  写这么一大坨的尴尬

- ```xml
  <typeAliases>
      <typeAlias type="com.pojo.User" alias="User"/>
  </typeAliases>
  ```

还可以指定一个包名，Mybatis会在指定的包下搜索Java Bean

- 扫描实体类的包，那么它默认的别名就是这个类的类名，首字母小写

- ```xml
  <typeAliases>
      <package name="com.pojo"/>
  </typeAliases>
  ```

还可以通过加注解的方式，就是在指定包下的实体类的类上加注解@Alias("hello")，那么此时它的别名为hello

### 映射器（mappers）

- 通过xml注册

```xml
<mappers>
    <mapper resource="com/dao/UserMapper.xml"/>
</mappers>
```

- 通过类注册

  - 注意

  - 接口和它的Mapper配置文件必须同名
  - 接口和它的Mapper配置文件必须在通一个包下

```xml
<mappers>
    <mapper class="com.dao.UserMapper"/>
</mappers>
```

- 通过包注册
  - 注意
  - 接口和它的Mapper配置文件必须同名
  - 接口和它的Mapper配置文件必须在通一个包下

```xml
<mappers>
    <package name="com.dao"/>
</mappers>
```

## 解决属性名和字段名不一致的问题

​		当pojo类里的属性值和数据库表里的字段名不一致的时候，由于查询语句返回的结果集不能和实体类对应上，并且我们输出的时候是输出的实体类，因此这些对应不上的属性就会输出null。

​		而解决的办法就是将属性值和字段使用resultMap映射

```xml
<resultMap id="map" type="hello">
    <result column="name" property="username"/>
</resultMap>

<select id="getUserById"  parameterType="int" resultMap="map">
    select * from mybatis.user where id = #{id};
</select>
```

次数resultMap里的type就是我们实体类使用注解加的别名

## 日志

### 日志工厂

```xml
<settings>
    <!--        标准的日志工程实现-->
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

### log4j

log4j.properties

```prop
log4j.rootLogger=DEBUG,console,file

#控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

#文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/kuang.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

### 可能遇到的问题

在生成日志文件后发现.log文件上出现蓝色问号，并且打不开该文件。

这是由于我们在mybatis-config.xml文件中设置类型别名时直接扫描的包，此时只要我们直接设置别名不扫描包，问题便可以迎刃而解

## @Param()注解

- 基本类型和String，需要加上（引用类型用实体类或者万能的map）
- 如果只有一个基本类型，可以选择不加，但是也是建议加上
- 我们在SQL中引用的就是@Param（）中设定的属性名

## lomobok

- 安装插件
- 导入jar包
- 在pojo上加注解
  - @Data：无参	get	set	toString	hashCode	equals	
  - @AllArgsConstructor：有参。但加上后会使无参消失
  - @NoArgsConstructor：无参。但加上后会使有参消失
    - 通常我们三个一起用

## 多对一查询处理

多个学生对应一个老师

- Student

  - ```java
    @Data
    public class Student {
        private int id;
        private String name;
        private Teacher teacher;
    }
    ```

- Teacher

  - ```java
    @Data
    public class Teacher {
        private int id;
        private String name;
    }
    ```

- 接口

  - ```java
    public interface StudentMapper {
        public List<Student> getStudent();
    }
    ```

- 实现类（StudentMapper.xml）【方式一：子查询】

  - ```xml
    <mapper namespace="com.dao.StudentMapper">
    <!--    先查出所有学生信息，再根据学生的tid查询老师-->
        <select id="getStudent" resultMap="Student">
            select * from student;
        </select>
        <resultMap id="Student" type="studentA">
            <result property="id" column="id"/>
            <result property="name" column="name"/>
    <!--        复杂的属性，我们需要单独处理   对象：association  集合：collection-->
            <association property="teacher" column="tid" javaType="teacherA" select="getTeacher"/>
        </resultMap>
        <select id="getTeacher" resultType="teacherA">
            select * from teacher where id = #{id};
        </select>
    </mapper>
    ```

    - 已经在mybatis-config.xml中将两个实体类映射成了studentA   teacherA

- 实现类（Mapper.xml）【方式一：按照结果集查询】

  - ```xml
    <mapper namespace="com.dao.StudentMapper">
    <!--    按照结果集查询-->
        <select id="getStudent2" resultMap="B">
            select s.id sid,s.name sname,t.name tname from student s,teacher t where s.tid=t.id;
        </select>
        <resultMap id="B" type="studentA">
            <result property="id" column="sid"/>
            <result property="name" column="sname"/>
            <association property="teacher" javaType="teacherA">
                <result property="name" column="tname"/>
            </association>
        </resultMap>
    </mapper>
    ```

    - 接口中多了方法getStudent2()

## 一对多查询处理

一个老师对应多个学生

- 学生类

  - ```java
    @Data
    public class Student {
        private int id;
        private String name;
        private int tid;
    }
    ```

- 老师类

  - ```java
    @Data
    public class Teacher {
        private int id;
        private String name;
        private List<Student> students;
    }
    ```

- 实现类（TeacherMapper.xml）

  - ```xml
    <mapper namespace="com.dao.TeacherMapper">
        <select id="getTeacher" resultMap="BB">
            select s.id sid,s.name sname,t.name tname,t.id tid from student s,teacher t where t.id=s.tid and tid=#{tid};
        </select>
        <resultMap id="BB" type="teacherA">
            <result property="id" column="tid"/>
            <result property="name" column="tname"/>
            <collection property="students" ofType="studentA">
                <result property="id" column="sid"/>
                <result property="name" column="sname"/>
            </collection>
        </resultMap>
    </mapper>
    ```

    - 在集合中的属性对应的类型应为ofType而不是javaType
      - javaType：用来写指定实体类中属性的类型
      - ofType：用来指定映射到List或者集合中的pojo类型，说白了就是泛型中的约束类型

## 动态SQl

**所谓动态SQL，就是指根据不同的条件生成不同的SQL语句**

- 生成唯一标识的工具类

  - ```java
    public static String getId(){
        return UUID.randomUUID().toString().replaceAll("-","");
    }
    ```

### IF

```xml
<select id="queryBlogIf" resultType="blog" parameterType="map">
    select * from blog where 1 = 1
    <if test="title != null">
        and title = #{title}
    </if>
    <if test="author != null">
        and author = #{author}
    </if>
</select>
```

### trim(where,set)

```xml
<select id="queryBlogIf" resultType="blog" parameterType="map">
    select * from blog
    <where>
        <if test="title != null">
            title = #{title}
        </if>
        <if test="author != null">
            and author = #{author}
        </if>
    </where>
</select>
```

- where：从前往后一次判断，哪个条件成立就走哪个条件语句
- set：和where用法一样。不过where用在查询语句中，而set用在插入语句中

### choose(when,otherwise)

```xml
<select id="queryBlogIf" resultType="blog" parameterType="map">
    select * from blog
    <where>
        <choose>
            <when test="title != null">
                title = #{title}
            </when>
            <when test="author != null">
                and author = #{author}
            </when>
            <otherwise>
                and views = #{views}
            </otherwise>
        </choose>
    </where>
</select>
```

- 就是相当于java中的switch case语句

### SQL片段

```xml
<sql id="xxx">
    #         这里写复用的代码
</sql>
#在用的地方用include标签来引用sql片段
<include refid="xxx"></include>
```

- sql片段是指将一些功能的部分抽取出来，方便复用。
- 注意事项
  - 最好基于单表来定义SQL语句，即定义简单的SQL语句放在SQL片段中
  - 片段里不要存在where标签

## 缓存

- 缓存原理：用户进来先找mapper中的二级缓存中找，如果没有，就进入sqlSession中，找一级缓存，如果还没有，才会查询数据库，然后将数据库中查到的数据存储在一级缓存中，当sqlSession关闭连接的时候，将数据存到二级缓存中去。

# ==SSM整合==

## 导入依赖

```xml
<!--    依赖：junit，数据库连接驱动，连接池，servlet，mybatis，mybatis-spring，spring-->
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.28</version>
        </dependency>
        <dependency>
            <groupId>c3p0</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.1.2</version>
        </dependency>
<!--        Servlet - JSP-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.1</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
<!--        mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.9</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.2</version>
        </dependency>
<!--        Spring-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.19.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.19.RELEASE</version>
        </dependency>

<!--        lombook-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
        </dependency>
    </dependencies>

<!--    静态资源导出问题-->
    <!--    在build中配置resources，来防止我们的资源导出失败的问题-->
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```

## 连接数据库

- 在idea右侧连接上数据库

## 写出java.main里的结构

- 多数为dao(mapper)	service	controller	pojo	filter	util	

## 写出resources里的资源文件

### jdbc.properties

```properties
jdbc.Driver=com.mysql.cj.jdbc.Driver
#使用MySql8.0以上，增加时区设置；   &serverTimezone=Asia/Shanghai
jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useSSL=ture&UniCode=ture&characterEncoding=utf-8
jdbc.username=root
jdbc.password=963724
```

### mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

<!--    配置数据源，交给Spring去做-->
    <typeAliases>
        <package name="com.pojo"/>
    </typeAliases>

<mappers>
    <mapper resource="/com/dao/BookMapper.xml"/>
</mappers>
</configuration>
```

- 在mybatis的核心配置文件中不再自己手写配置数据源的工作，交给spring去做
- 这里给指定包下的类起别名（这里指pojo）。方便后面写mapper.xml

## 根据数据库将pojo写出来

- 注意字段要与实体类中的属性一一对应

## 写dao（mapper）中实体类对应的接口

- 接口中的方法传递参数时记得加上@Param("")参数

## 写mapper.xml

- 注意一个接口对应一个mapper中的namespace
- 里面写对应的实现方法（CRUD）

## 在mybatis-config中注册绑定mapper.xml

- 见4.4.2

## 写Service层

- service中的接口的内容与dao层的接口内容大致相同
- 写serviceImpl
  - servce调用dao
  - 在impl中先创建属性dao层的mapper
  - 在每一个重写的方法的返回值处改为mapper.dao层接口的方法

## 开始用Spring整合

### Spring-dao

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd

">
    
<!--    关联数据库配置文件-->
    <context:property-placeholder location="classpath:database.properties"/>
    
    
<!--    连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.Driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    
    
<!--    sqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
<!--        引入数据源-->
        <property name="dataSource" ref="dataSource"/>
<!--        关联mybatis配置文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
    </bean>
    
    
<!--    配置dao接口扫描包，动态的是实现dao接口可以注入到Spring容器中-->
<!--    MapperScannerConfigurer扫描包下的所有接口，并将每个接口执行getMapper()方法，获得每个dao对象-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
<!--        注入sqlSessionFactory-->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
<!--        要扫描的dao包-->
        <property name="basePackage" value="com.dao"/>
    </bean>
</beans>
```

- 关联数据库配置文件
- 注入数据源（以c3p0为例）
- 注入sqlSessionFactory
  - 这里面需要引入数据源
  - 关联mybatis-config核心配置文件
- 配置扫描包（使dao接口注入到Spring容器中）

### Spring-service

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd

">

<!--    扫描service下的包-->
    <context:component-scan base-package="com.service"/>

<!--    将所有业务类注入到Spring-->
    <bean id="BookServiceImpl" class="com.service.BookServiceImpl">
        <property name="bookMapper" ref="bookMapper"/>
    </bean>

<!--    声明式事务配置-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<!--        注入数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

<!--    aop事务支持-->
</beans>
```

- 扫描service下的包
- 将所有的业务类（impl）注入到Spring容器中
- 声明式事务配置
  - 在里面注入数据源
- 【aop事务支持】

### spring-mvc

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd

">

<!--    注解驱动-->
<!--    静态资源过滤-->
<!--    扫描包 controller-->
    <mvc:annotation-driven/>
    <mvc:default-servlet-handler/>
    <context:component-scan base-package="com.controller"/>

<!--    视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

- 新建webapp资源文件
- 在WEB-INF下创建jsp文件夹（与视图解析器中的前缀后缀匹配上）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

<!--    前端控制器 DispatchServlet-->
    <servlet>
        <servlet-name>applicationContext</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>applicationContext</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

<!--    乱码解决-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

<!--    Session-->
    <session-config>
        <session-timeout>15</session-timeout>
    </session-config>
</web-app>
```

- 在web.xml中设置前端控制器   解决乱码的问题

## 连接整合的三层spring(applicationContext.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    <import resource="spring-dao.xml"/>
    <import resource="spring-service.xml"/>
    <import resource="spring-mvc.xml"/>
</beans>
```







