## 1、安全：

Spring Security是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型。他可以实现强大的web安全控制。对于安全控制，我们仅需引入spring-boot-starter-security模块，进行少量的配置，即可实现强大的安全管理。

 几个类：

WebSecurityConfigurerAdapter：自定义Security策略

AuthenticationManagerBuilder：自定义认证策略

@EnableWebSecurity：开启WebSecurity模式

* 应用程序的两个主要区域是“认证”和“授权”（或者访问控制）。这两个主要区域是Spring Security 的两个目标。

* “认证”（Authentication），是建立一个他声明的主体的过程（一个“主体”一般是指用户，设备或一些可以在你的应用程序中执行动作的其他系统）。

* “授权”（Authorization）指确定一个主体是否允许在你的应用程序执行一个动作的过程。为了抵达需要授权的店，主体的身份已经有认证过程建立。

* 这个概念是通用的而不只在Spring Security中。

## 2、Web&安全

1.登陆/注销

​	–HttpSecurity配置登陆、注销功能

2.Thymeleaf提供的SpringSecurity标签支持

​	–需要引入thymeleaf-extras-springsecurity4

​	–sec:authentication=“name”获得当前用户的用户名

​	–sec:authorize=“hasRole(‘ADMIN’)”当前用户必须拥有ADMIN权限时才会显示标签内容

3.remember me

​	–表单添加remember-me的checkbox

​	–配置启用remember-me功能

4.CSRF（Cross-site request forgery）跨站请求伪造

​	HttpSecurity启用csrf功能，会为表单添加_csrf的值，提交携带来预防CSRF；

## 3、SpringBoot整合

官方参考地址：

https://docs.spring.io/spring-security/site/docs/current/guides/helloworld-boot.html

### 1、引入POM

```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 2、编写SpringSecurity的配置类

```java
@EnableWebSecurity
public class MySecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //super.configure(http);
        //定制请求的授权规则
        http.authorizeRequests().antMatchers("/").permitAll()
                .antMatchers("/level1/**").hasRole("VIP1")
                .antMatchers("/level2/**").hasRole("VIP2")
                .antMatchers("/level3/**").hasRole("VIP3");

        //开启自动配置的登陆功能，效果，如果没有登陆，没有权限就会来到登陆页面
        http.formLogin().usernameParameter("user").passwordParameter("pwd")
                .loginPage("/userlogin");
        //1、/login来到登陆页
        //2、重定向到/login?error表示登陆失败
        //3、更多详细规定
        //4、默认post形式的 /login代表处理登陆
        //5、一但定制loginPage；那么 loginPage的post请求就是登陆


        //开启自动配置的注销功能。
        http.logout().logoutSuccessUrl("/");//注销成功以后来到首页
        //1、访问 /logout 表示用户注销，清空session
        //2、注销成功会返回 /login?logout 页面；

        //开启记住我功能
        http.rememberMe().rememberMeParameter("remeber");
        //登陆成功以后，将cookie发给浏览器保存，以后访问页面带上这个cookie，只要通过检查就可以免登录
        //点击注销会删除cookie

    }

    //定义认证规则
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //super.configure(auth);
        auth.inMemoryAuthentication()
                .withUser("zhangsan").password("123456").roles("VIP1","VIP2")
                .and()
                .withUser("lisi").password("123456").roles("VIP2","VIP3")
                .and()
                .withUser("wangwu").password("123456").roles("VIP1","VIP3");

    }
}
```

### 3、thymeleaf与springsecurity整合

1、POM

```
		<dependency>
			<groupId>org.thymeleaf.extras</groupId>
			<artifactId>thymeleaf-extras-springsecurity5</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
```

2、引入名称空间

```
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org" xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
```

3、页面：

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
     xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1 align="center">欢迎光临武林秘籍管理系统</h1>
<div sec:authorize="!isAuthenticated()">
   <h2 align="center">游客您好，如果想查看武林秘籍 <a th:href="@{/userlogin}">请登录</a></h2>
</div>
<div sec:authorize="isAuthenticated()">
   <h2><span sec:authentication="name"></span>，您好,您的角色有：
      <span sec:authentication="principal.authorities"></span></h2>
   <form th:action="@{/logout}" method="post">
      <input type="submit" value="注销"/>
   </form>
</div>

<hr>

<div sec:authorize="hasRole('VIP1')">
   <h3>普通武功秘籍</h3>
   <ul>
      <li><a th:href="@{/level1/1}">罗汉拳</a></li>
      <li><a th:href="@{/level1/2}">武当长拳</a></li>
      <li><a th:href="@{/level1/3}">全真剑法</a></li>
   </ul>

</div>

<div sec:authorize="hasRole('VIP2')">
   <h3>高级武功秘籍</h3>
   <ul>
      <li><a th:href="@{/level2/1}">太极拳</a></li>
      <li><a th:href="@{/level2/2}">七伤拳</a></li>
      <li><a th:href="@{/level2/3}">梯云纵</a></li>
   </ul>

</div>

<div sec:authorize="hasRole('VIP3')">
   <h3>绝世武功秘籍</h3>
   <ul>
      <li><a th:href="@{/level3/1}">葵花宝典</a></li>
      <li><a th:href="@{/level3/2}">龟派气功</a></li>
      <li><a th:href="@{/level3/3}">独孤九剑</a></li>
   </ul>
</div>
</body>
</html>
```