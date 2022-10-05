# yaml

## 给属性赋值

- 首先创建一个pojo类并自定义属性
- 在yaml文件中写下对应的赋值
- 在pojo的类头写如下注解

```java
@ConfigurationProperties(prefix = "person")
```

- 此时我们的所有值都将赋到里面

- 写@ConfigurationProperties时编译器会报红，此时需要根据官方提示导入依赖

- ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-configuration-processor</artifactId>
  </dependency>
  ```

## 松散绑定

如last-name==>lastName

简单的说就是在yaml配置中写下xxx-xxx在pojo属性中可以使用小驼峰接收

# JSR303校验

- 导入依赖

- ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
  ```

- 在类上开启注解@Validated

- 在属性上加类似@Email(message="邮箱格式错误")的注解

- 此时，当我们给属性赋值的时候，赋错了格式，那么将直接给你报错

- 其他的格式见百度jsr303检验

# Thymeleaf

导入依赖

```xml
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-java8time</artifactId>
</dependency>
```

- 文件为.html文件，位置放在classpath:templates文件夹下

- 导入头文件

- ```html
  <html lang="en" xmlns:th="http://www.thymeleaf.org">
  ```

- 链接

  - ```html
     th:href="@{/css/bootstrap.min.css}
    ```

- 文本

  - ```html
    th:text="#{login.remember}"
    ```

- 所有的html元素都可以被thymleaf接管

# 扩展SpringMVC

- ```java
  package com.zhao.config;
  
  import org.springframework.context.annotation.Configuration;
  import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
  import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
  
  @Configuration
  public class MyMvcConfig implements WebMvcConfigurer {
  
      @Override
      public void addViewControllers(ViewControllerRegistry registry) {
          registry.addViewController("/zhaolong").setViewName("test");
  
      }
  }
  ```

  - 首先添加注解@Configuration
  - 其次该类实现接口WebMvcConfigurer

- 如上图案例中，我们访问/zhaolong，就会自动跳转到test.html页面

# i18n

- 国际化

  - 在配置文件中配置文件的真实位置

  - ```yaml
    spring:
      messages:
        #我们配置文件的真实位置
        basename: i18n.login
    ```

  - 在资源文件中要用#{}来写下自己要替换的内容
  
- 我们可以配置自己的国际化试图控制

  - ```java
    public class MylocaleResolver implements LocaleResolver {
    
        @Override
        public Locale resolveLocale(HttpServletRequest request) {
    
            //获取客户端的请求参数
            String language = request.getParameter("lan");
            System.out.println(language);
            Locale locale = Locale.getDefault();
            //如果请求的链接携带了请求参数
            if (StringUtils.hasText(language)) {
                String[] split = language.split("_");
                //返回 国家 地区
                locale = new Locale(split[0], split[1]);
            }
            return locale;
        }
    
        @Override
        public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
    
        }
    }
    ```

- 然后在配置类中将我们自定义的类注入到Spring容器中

  - ```java
    @Bean
        public LocaleResolver localeResolver(){
            return new MylocaleResolver();
        }
    ```


# SpringBoot创建项目

- html页面放在templates目录下

- img、css、js等文件放在static文件夹下

- 404、500页面放在error页面下（当程序出现这类异常后，springboot会自动到error文件夹下寻找这些页面）

- ==tip：所有页面的静态资源都需要使用thymeleaf接管。==

- 创建数据库、编写实体类

  - 注意实体类属性的命名规范（使用驼峰，尽量不加"__"）

- 配置yaml文件

  - 在这里可以

  - ```yaml
    server:
      servlet:
        #应用上下文位置
        context-path: /zhao
    spring:
      thymeleaf:
        #关闭thymeleaf默认缓存
        cache: false
      messages:
        #我们配置文件的真实位置
        basename: i18n.login
      mvc:
        format:
          date: yyyy-MM-dd
        #配置数据源  
        datasource:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://127.0.0.1:3306/z_login?useUnicode=true&characterEncoding=UTF-8&useSSL=false&useAffectedRows=true
        username: root
        password: 963724
    #配置mybatis
    mybatis-plus:
      type-aliases-package: com.pojo
      mapper-locations: classpath:mybatis/mapper/*.xml
      global-config:
        db-config:
          id-type: auto
    ```

- MVC三层架构写出来

- mapper.xml写在resources文件夹下的mybatis文件夹里

  - 在yaml中我们已经配置了mapper-location：classpath:mybatis/mapper/*.xml

- 写出业务逻辑

- 写MVC配置类

  - ```java
    package com.zhao.config;
    
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.servlet.LocaleResolver;
    import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
    import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
    
    @Configuration
    public class MyMvcConfig implements WebMvcConfigurer {
    	//添加视图控制
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            registry.addViewController("/").setViewName("index");
            registry.addViewController("/index.html").setViewName("index");
            registry.addViewController("/main.html").setViewName("dashboard");
    
        }
    	//添加本地解析
        @Bean
        public LocaleResolver localeResolver(){
            return new MylocaleResolver();
        }
    	//配置拦截器
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new LoginHandlerInterceptor()).
                    addPathPatterns("/**").
                    excludePathPatterns("/index.html","/","/user/login","/css/**","/img/**","/js/**");
        }
    }
    ```

    - 自定义拦截器

    - ```java
      package com.zhao.config;
      
      import org.springframework.web.servlet.HandlerInterceptor;
      
      import javax.servlet.http.HttpServletRequest;
      import javax.servlet.http.HttpServletResponse;
      
      public class LoginHandlerInterceptor implements HandlerInterceptor {
      
          @Override
          public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
      
              Object loginUser = request.getSession().getAttribute("loginUser");
      
              if (loginUser == null){
                  request.setAttribute("msg","没有权限，请先登录");
                  request.getRequestDispatcher("/index.html").forward(request,response);
                  return false;
              }else {
                  return true;
              }
          }
      }
      
      ```

    - 自定义本地解析（中英文切换）

    - ```java
      package com.zhao.config;
      
      import org.springframework.util.StringUtils;
      import org.springframework.web.servlet.LocaleResolver;
      
      import javax.servlet.http.HttpServletRequest;
      import javax.servlet.http.HttpServletResponse;
      import java.util.Locale;
      
      
      public class MylocaleResolver implements LocaleResolver {
      
          @Override
          public Locale resolveLocale(HttpServletRequest request) {
      
              //获取客户端的请求参数
              String language = request.getParameter("lan");
              Locale locale = Locale.getDefault();
              //如果请求的链接携带了请求参数
              if (StringUtils.hasText(language)) {
                  String[] split = language.split("_");
                  //返回 国家 地区
                  locale = new Locale(split[0], split[1]);
              }
              return locale;
          }
      
          @Override
          public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
      
          }
      }
      
      ```

- 此时，大功告成

# SpringSecurity（安全）

在web开发中，安全第一位。过滤器、拦截器

```java
package com.zhao.config;

import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //添加请求授权规则
        http.authorizeHttpRequests()
                .antMatchers("/").permitAll()
                .antMatchers("/level1/**").hasRole("vip1")
                .antMatchers("/level2/**").hasRole("vip2")
                .antMatchers("/level3/**").hasRole("vip3");
        //如果没有授权，会跳转到登录页面
        // /login
        //定制登录页面loginPage("/toLogin")
        http.formLogin().loginPage("/toLogin").usernameParameter("username")
        .passwordParameter("password")
        .loginProcessingUrl("/login");
        //开启注销功能    注销成功，返回首页
        http.logout().logoutSuccessUrl("/");
        //防止网站攻击    get、post
        http.csrf().disable();//关闭csrf（跨站请求伪造）功能
        http.logout().logoutSuccessUrl("/");
        //开启记住我功能   cookie，默认保存，14天     自定义接收前端的参数
        http.rememberMe().rememberMeParameter("remember");
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())//设置好了加密方式
                .withUser("u1").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2")
                .and()
                .withUser("u2").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1")
                .and()
                .withUser("u3").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2","vip3");
    }
}
```

- 这个类必须要继承 WebSecurityConfigurerAdapter 类

# Swagger

新建一个SpringBoot-Web项目

导入相关依赖

```xml
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>3.0.0</version>
</dependency>
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>3.0.0</version>
</dependency>

```

编写一个HelloWorld工程

配置Swagger==>Config

```java
package com.zhao.swaggerdemo.config;

import org.springframework.context.annotation.Configuration;
import springfox.documentation.oas.annotations.EnableOpenApi;

@Configuration
@EnableOpenApi
public class SwaggerConfig {
}
```

```yaml
spring:
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher
```

配置Swagger



# 发送邮件

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

```java
spring:
  mail:
    username: 1976915593@qq.com
    password: wpqlmlgagfoxbbci
    #配置服务主机
    host: smtp.qq.com
    #开启加密验证   QQ特有的
    properties:
      mail:
        smtp:
          ssl:
            enable=true
        debug: true
```

```java
package com.zhao;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSenderImpl;
import org.springframework.mail.javamail.MimeMessageHelper;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import java.io.File;

@SpringBootTest
class DemoApplicationTests {

    @Autowired
    JavaMailSenderImpl javaMailSender;//先将发送邮件的类自动注入到Spring容器中

    @Test
    void contextLoads() {
        //一个简单的邮件
        SimpleMailMessage mimeMessage = new SimpleMailMessage();

        mimeMessage.setSubject("赵龙");//设置邮件标题
        mimeMessage.setText("人生不过短短3万天 ");//设置邮件内容
        mimeMessage.setTo("1976915593@qq.com");
        mimeMessage.setFrom("1976915593@qq.com");

        javaMailSender.send(mimeMessage);
    }

    @Test
    void contextLoads2() throws MessagingException {
        //一个复杂的邮件
        MimeMessage mimeMessage = javaMailSender.createMimeMessage();
        //创建复杂信息的第二方式   new出来
        //MimeMessage mimeMessage1 = new MimeMessage();
        //组装    true代表开启multipart多文件支持
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,true);
        //正文
        helper.setSubject("赵龙");
        helper.setText("<p style='color:red'>人生不过短短3万天</p>",true);//true代表开启html支持

        //附件
        helper.addAttachment("1.jpg",new File("C:\\Users\\zhao\\Desktop\\1.jpg"));
        helper.addAttachment("2.jpg",new File("C:\\Users\\zhao\\Desktop\\1.jpg"));

        helper.setTo("1976915593@qq.com");
        helper.setFrom("1976915593@qq.com");
        javaMailSender.send(mimeMessage);
    }
}
```

# 定时执行任务

```java
package com.zhao;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication

@EnableScheduling//开启定时功能的注解
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

```java
package com.zhao.service;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

@Service
public class ScheduledService {
    //cron表达式
    //秒 分 时 日 月 周几
    /**
     * 30 54 16 * * 0-7     每天的16：54:30执行一次
     * 30 0/5 10,18 ** ？    每天的10点和18点执行一次，每隔5分钟执行一次
     * 
     */
    @Scheduled(cron = "30 54 16 * * 0-7")
    public void hello(){
        System.out.println("hello，赵龙");
    }
}
```

- 首先在启动类上标注注解@EnableScheduling，开启定时功能的注解
-  再在业务类上加上这个注解@Scheduled(cron = "30 54 16 * * 0-7")【以上代码详细讲解了】
- 关于cron表达式，可以自行百度解决

