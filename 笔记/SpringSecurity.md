# SpringSecurity

> 2021.09.11 王飞

## 1. SpringSecurity介绍

web安全的两个核心

- 用户认证：判断用户能否登录
- 用户授权：判断用户是否有权限去做某些事情（权限）

## 2. SpringSecurity入门案例

### 2.1 创建SpringBoot工程
### 2.2 导入依赖

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
~~~

### 2.3 配置

只需新建配置类并添加 @EnableWebSecurity、@Configuration 两个注解

~~~java
@EnableWebSecurity
@Configuration
public class SpringSecurityConfiguration extends WebSecurityConfigurerAdapter {
​
}
~~~

- 默认用户名密码

  默认用户名：user

  默认密码：（在控制台打印出来）<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210913163951.png" alt="image-20210913163951060" style="zoom:50%;" />

- 静态用户名密码配置

  在Springboot的配置文件中添加

~~~properties
spring:
  security:
    user:
      name: zhangsan
      password: 123456
~~~



### 2.4 编写Controller测试

编写相应的controller，启动工程，访问路径http://localhost:8080/，跳转到登录页面

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210913163822.png" alt="image-20210913163822672" style="zoom:50%;" />

输入配置的用户名和密码登录

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921122636.png" alt="image-20210921122636838" style="zoom:50%;" />

## 3. 自定义登录页

步骤：

1. 新建登录页面

   注意表单中的用户名密码名必须叫username和password，因为源码中写死了这两个名字。

   <img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921141234.png" alt="image-20210921141234595" style="zoom:50%;" />

   ~~~html
   <!DOCTYPE html>
   <html xmlns:th="http://www.thymeleaf.org">
   <head>
       <meta charset="UTF-8">
       <title>登录-SpringSecurity</title>
   </head>
   <body align="center">
       <form action="/login" method="post" th:action="@{/login}" th:method="post">
           <div>
               <input type="text" name="username" placeholder="用户名">
           </div>
           <div>
               <input type="password" name="password" placeholder="密码">
           </div>
           <div class="checkbox">
               <label><input type="checkbox"> 记住我</label>
           </div>
           <button type="submit">登录</button>
           <p align="text-center"> <a href="login.html#">
               <small>忘记密码了？</small></a> | <a href="#">注册一个新账号</a>
           </p>
       </form>
   </body>
   </html>
   ~~~

2. 在SpringSecurity的配置类中增加配置选项

   ~~~java
   http.formLogin().loginPage("/login").permitAll()
   ~~~

   设置登录页的路径为/login ，并允许所有请求访问。

3. 编写controller，当请求/login时返回登录页面

   ~~~java
   @Controller
   public class LoginController {
   
       @RequestMapping("/login")
       public String login() {
           return "login";
       }
   
   }
   ~~~

启动项目，会跳转到自定义登录页

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921141749.png" alt="image-20210921141749511" style="zoom:50%;" />

## 4. 配置不拦截静态资源

在html中加样式会被SpringSecurity拦截。

若要配置不拦截，

只需在配置类中覆写**configure(WebSecurity web)** 方法，添加忽略路径格式 

~~~java
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/css/**", "/js/**", "/plugins/**", "/images/**", "/fonts/**");
    }
~~~



## 5. 登录成功页面

登录成功后跳转到指定页面。

在配置类中增加配置项

~~~java
http.formLogin().loginPage("/login").defaultSuccessUrl("/index").permitAll()
~~~

这种配置系统并不会始终跳转到指定页面，只有访问根路径的链接时才会跳转。

如果需要任何时候登录成功都跳转指定页面，需要如下配置：

~~~Java
http.formLogin().loginPage("/login").defaultSuccessUrl("/user/index", true).permitAll()
~~~

该部分逻辑引用流程图表示(https://blog.csdn.net/liuminglei1987/article/details/106936595)

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921143742.jpeg" alt="img" style="zoom: 67%;" />

## 6. 成功登录的附加操作

登录成功后需要记录，存数据库、记日志或发消息。可以使用登录成功的 **SuccessHandler** 进行设置。

步骤：

1. 自定义类CustomSavedRequestAwareAuthenticationSuccessHandler重写SavedRequestAwareAuthenticationSuccessHandler的onAuthenticationSuccess()方法。

   ~~~java
       @Override
       public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws ServletException, IOException {
           super.onAuthenticationSuccess(request, response, authentication);
   
           this.logger.info(String.format("IP %s，用户 %s， 于 %s 成功登录系统。", request.getRemoteHost(), authentication.getName(), LocalDateTime.now()));
   
           try {
               // 发邮件
               this.emailService.send();
   
               // 发短信
               this.smsService.send();
   
               // 发微信
               this.weChatService.send();
           } catch (Exception ex) {
               this.logger.error(ex.getMessage(), ex);
           }
       }
   ~~~

2. 编写service，分别用于发邮件等操作

   <img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921145842.png" alt="image-20210921145841951" style="zoom:50%;" />

3. 在配置类中增加配置项

   ~~~java
   .successHandler(customAuthenticationSuccessHandler())
   ~~~

   以及

   ~~~java
       @Bean
       public AuthenticationSuccessHandler customAuthenticationSuccessHandler() {
           CustomSavedRequestAwareAuthenticationSuccessHandler customSavedRequestAwareAuthenticationSuccessHandler = new CustomSavedRequestAwareAuthenticationSuccessHandler();
           customSavedRequestAwareAuthenticationSuccessHandler.setDefaultTargetUrl("/index");
           customSavedRequestAwareAuthenticationSuccessHandler.setEmailService(emailService);
           customSavedRequestAwareAuthenticationSuccessHandler.setSmsService(smsService);
           customSavedRequestAwareAuthenticationSuccessHandler.setWeChatService(wechatService);
           return customSavedRequestAwareAuthenticationSuccessHandler;
       }
   ~~~

启动项目，登录成功后可以看到执行了自定义的service中的方法

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921145946.png" alt="image-20210921145946213" style="zoom:50%;" />

## 7. SpringSecurity配置类的详细说明

配置类中的配置项比较多。

需要注意的一点事配置项是**有先后顺序的，先配置的先生效**

## 8. 用户登出

**Spring Security** 用户登出配置中，**CSRF** 默认是开启的

> CSRF (Cross Site Request Forgery)
>
> 中文名：跨站请求伪造
>
> 是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。

> XSS ( Cross-site scripting ) 
>
> 中文名：跨站脚本

需要关闭csrf

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921150804.png" alt="image-20210921150804017" style="zoom:50%;" />

### 8.1 用户登出成功重定向url

**Spring Security** 框架默认的用户登出成功url为 **/login?logout**

可以自定义登出成功的url

配置类中增加配置项

~~~java
http.logout().logoutSuccessUrl("/logout_success").permitAll()
~~~

登出的controller

~~~java
@Controller
public class LogoutController {

    @RequestMapping("/logout_success")
    public String logoutSuccess() {
        return "logout_success";
    }

}
~~~

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921152812.png" alt="image-20210921152812912" style="zoom:50%;" />

### 8.2 登出成功**LogoutSuccessHandler**接口

此接口中的方法onLogoutSuccess()可以实现自定义的登出成功后操作

可以发邮件、记日志等操作



## 9. 自定义登录失败页面

**Spring Security** 框架自带的登录失败url为 **/login?error**

可以在配置类中增加配置项，指定登录失败页面

![image-20210921165024670](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921165024.png)

然后编写controller

~~~java
    @RequestMapping("/login_fail")
    public String loginFail(HttpServletRequest request, Model model) {
        return "login_fail";
    }
~~~

自定义登录失败页面

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921165206.png" alt="image-20210921165206033" style="zoom:50%;" />

启动项目，登录失败会跳转到指定的页面

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921165333.png" alt="image-20210921165333701" style="zoom:50%;" />

登录失败同样可以进行一些附加处理，实现接口**AuthenticationFailureHandler**即可

 **url区分不同的登录失败场景**



常用的登录失败枚举

~~~Java
FAILURE(0, "登录失败！"),
​
BADCREDENTIALS(1, "用户名密码错误！"),
​
LOCKED(2, "用户已被锁定，无法登录！"),
​
ACCOUNTEXPIRED(3, "用户已过时，无法登录！"),
​
USERNAMENOTFOUND(4, "用户不存在！");
~~~

详情参考https://blog.csdn.net/liuminglei1987/article/details/107363408

## 10. @PreAuthorize

可以实现权限控制

使用时需要开启权限控制注解，在配置类中开启@EnableGlobalMethodSecurity(prePostEnabled = true)

~~~java
@EnableWebSecurity
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SpringSecurityConfiguration extends WebSecurityConfigurerAdapter {
...
}
~~~

然后即可在需要权限控制的controller方法上加上权限注解@PreAuthorize

~~~java
    @PreAuthorize("hasRole('User')")
    @RequestMapping("/index")
    public String index() {
        return "/user/index";
    }
~~~

**该注解的源码与参数含义：**

~~~java
@Target({ ElementType.METHOD, ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface PreAuthorize {
	/**
	 * @return the Spring-EL expression to be evaluated before invoking the protected
	 * method
	 */
	String value();
}
~~~

**value** （可省略。如"hasRole('User')"）属性是 **Spring-EL** 表达式类型的字符串

参数值（即EL表达式字符串）可以是**SecurityExpressionRoot**类中的方法

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921170800.png" alt="image-20210921170800205" style="zoom:50%;" />

常用的方法有：

- **hasRole**，对应 public final boolean hasRole(String role) 方法，含义为必须含有某角色（非ROLE_开头），如有多个的话，必须同时具有这些角色，才可访问对应资源。

- **hasAnyRole**，对应 public final boolean hasAnyRole(String... roles) 方法，含义为只具有有某一角色（多多个角色的话，具有任意一个即可），即可访问对应资源。

- **hasAuthority**，对应 public final boolean hasAuthority(String authority) 方法，含义同 hasRole，不同点在于这是权限，而不是角色，区别就在于权限往往带有前缀（如默认的ROLE_），而角色只有标识。

- **hasAnyAuthority**，对应 public final boolean hasAnyAuthority(String... authorities) 方法，含义同 hasAnyRole，不同点在于这是权限，而不是角色，区别就在于权限往往带有前缀（如默认的ROLE_），而角色只有标识

- **permitAll**，对应 public final boolean permitAll() 方法，含义为允许所有人（可无任何权限）访问。

- **denyAll**，对应 public final boolean denyAll() 方法，含义为不允许任何（即使有最大权限）访问。

- **isAnonymous**，对应 public final boolean isAnonymous() 方法，含义为可匿名（不登录）访问。

- **isAuthenticated**，对应 public final boolean isAuthenticated() 方法，含义为身份证认证后访问。

- **isRememberMe**，对应 public final boolean isRememberMe() 方法，含义为记住我用户操作访问。

- **isFullyAuthenticated**，对应 public final boolean isFullyAuthenticated() 方法，含义为非匿名且非记住我用户允许访问。

![image-20210921171240839](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921171240.png)

启动项目，访问加了权限注解的controller，当前用户没有权限时会返回403

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921171622.png" alt="image-20210921171622125" style="zoom:50%;" />

给之前额用户设置角色User

~~~properties
spring:
  security:
    user:
      name: bei_zhang
      password: 111111
      roles:
        - User
~~~

再次访问，即可成功。

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921171837.png" alt="image-20210921171837533" style="zoom:50%;" />

## 11. 动态用户（数据库查询用户）

动态用户即通过数据库查询并验证用户名与密码

其实现方式是自定义UserDetailsService

这里使用mybatis访问数据库，数据库使用mysql

步骤：

1. 引入依赖

   ~~~xml
           <dependency>
               <groupId>com.baomidou</groupId>
               <artifactId>mybatis-plus-boot-starter</artifactId>
               <version>3.4.3</version>
           </dependency>
   
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>8.0.11</version>
           </dependency>
   
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
           </dependency>
   ~~~

   

2. 创建数据库和表

   ![image-20210921181215021](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921181215.png)

3. 创建实体类和mapper层

   <img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921181134.png" alt="image-20210921181134533" style="zoom:50%;" />

   使用mybatisPluse，只需要新建mapper层的接口并继承BaseMapper，且在泛型中传入对应的实体类即可。BaseMapper中已经实现了增删改查方法，可以直接调用。

   <img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921175602.png" alt="image-20210921175602117" style="zoom:50%;" />

   

4. 在项目配置文件中指定数据库的驱动地址和用户名密码

   <img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921175452.png" alt="image-20210921175452666" style="zoom:50%;" />

   

5. 实现UserDetailsService

   ~~~java
   @Service("userDetailsService")
   public class MyUserDetailsService implements UserDetailsService {
   
       @Autowired
       SysUserMapper mapper;
   
       @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
           QueryWrapper<SysUser> wrapper = new QueryWrapper<>();
           wrapper.eq("username",username);
           SysUser sysUser = mapper.selectOne(wrapper);
           if (sysUser == null){
               // 数据库中没有该用户，失败
               throw new UsernameNotFoundException("用户名不存在");
           }
           // 设置用户的权限字符串
           List<GrantedAuthority> authorities = AuthorityUtils.commaSeparatedStringToAuthorityList(sysUser.getAuth());
           return new User(sysUser.getUsername(),new BCryptPasswordEncoder().encode(sysUser.getPassword()),authorities);
       }
   }
   ~~~

   

   <img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921181258.png" alt="image-20210921181257968" style="zoom:50%;" />

   返回的User对象是SpringSecurity中的用户详情。

6. 在启动类上加上mapper包扫描

   ~~~java
   @MapperScan("com.wf.securetystudy.mapper")
   ~~~

7. 在配置类中增加配置项，配置使用的UserDetailsService，并指定密码加密方式

   ~~~java
       @Autowired
       MyUserDetailsService userDetailsService;
       
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           auth.userDetailsService(userDetailsService).passwordEncoder(new BCryptPasswordEncoder());
       }
   ~~~

启动项目，访问具有权限控制的controller路径

使用具有admin权限字符串的bei_zhang用户登录能够成功

使用没有权限字符串的mary用户登录显示403

## 12. FilterSecurityInterceptor

### 12.1  FilterSecurityInterceptor方法详解



**Spring Security Filter Chain** 的最后一个 **Filter**

可以**获取当前 request 对应的权限配置**，首先是调用基类的 **beforeInvocation** 方法

~~~java
public void invoke(FilterInvocation fi) throws IOException, ServletException {
    if ((fi.getRequest() != null)
        ......
    }
    else {
        ......
        InterceptorStatusToken token = super.beforeInvocation(fi);
        ......
    }
}
~~~

其基类为**AbstractSecurityInterceptor**，基类的**beforeInvocation**方法从**SecurityMetadataSource**中获取当前request对应的**ConfigAttribute**，即权限信息

~~~java
protected InterceptorStatusToken beforeInvocation(Object object) {
    ......
    Collection<ConfigAttribute> attributes = this.obtainSecurityMetadataSource()
        .getAttributes(object);
    if (attributes == null || attributes.isEmpty()) {
        if (rejectPublicInvocations) {
            throw new IllegalArgumentException(
                "Secure object invocation "
                + object
                + " was denied as public invocations are not allowed via this interceptor. "
                + "This indicates a configuration error because the "
                + "rejectPublicInvocations property is set to 'true'");
        }
        ......
    }
}
~~~

 **rejectPublicInvocations** 属性，默认为 **false**。此属性含义为**拒绝公共请求**。

接着判断是否需要进行身份认证，调用**authenticateIfRequired**方法

~~~java
protected InterceptorStatusToken beforeInvocation(Object object) {
    ......
​
    Authentication authenticated = authenticateIfRequired();
​
    ......
}
~~~

- 首先判断当前用户是否已通过身份认证，如已通过，则直接返回；
- 如尚未通过身份认证，则调用身份认证管理器**AuthenticationManager**认证；认证通过会在当前安全上下文中存储认证后的**authentication**；

~~~java
private Authentication authenticateIfRequired() {
    Authentication authentication = SecurityContextHolder.getContext()
        .getAuthentication();
    if (authentication.isAuthenticated() && !alwaysReauthenticate) {
        if (logger.isDebugEnabled()) {
            logger.debug("Previously Authenticated: " + authentication);
        }
        return authentication;
    }
    authentication = authenticationManager.authenticate(authentication);
    // We don't authenticated.setAuthentication(true), because each provider should do
    // that
    if (logger.isDebugEnabled()) {
        logger.debug("Successfully Authenticated: " + authentication);
    }
    SecurityContextHolder.getContext().setAuthentication(authentication);
    return authentication;
}
~~~

然后继续在**beforeInvocation**方法中执行**AccessDecisionManager**的**decide**方法对当前请求进行鉴权。

> 认证+鉴权  两个功能

~~~java
protected InterceptorStatusToken beforeInvocation(Object object) {
    ......
    // Attempt authorization
    try {
          this.accessDecisionManager.decide(authenticated, object, attributes);
        }
    catch (AccessDeniedException accessDeniedException) {
        publishEvent(new AuthorizationFailureEvent(object, attributes, authenticated,
                                                   accessDeniedException));
        throw accessDeniedException;
    }
    if (debug) {
        logger.debug("Authorization successful");
    }
    if (publishAuthorizationSuccess) {
        publishEvent(new AuthorizedEvent(object, attributes, authenticated));
    }
}
~~~

无论鉴权通过或是不通后，**Spring Security 框架均使用了观察者模式**，来通知其它Bean，当前请求的鉴权结果。

- 如果鉴权不通过，则会抛出 **AccessDeniedException** 异常，即访问受限，然后会被 **ExceptionTranslationFilter** 捕获，最终解析后调转到对应的鉴权失败页面。
- 如果鉴权通过，**AbstractSecurityInterceptor** 通常会继续请求。

在 **AccessDecisionManager** 鉴权成功后，将通过 **RunAsManager** 在现有 **Authentication** 基础上构建一个新的**Authentication**，如果新的 **Authentication** 不为空则将产生一个新的 **SecurityContext**，并把新产生的**Authentication** 存放在其中。这样在请求受保护资源时从 **SecurityContext**中 获取到的 **Authentication** 就是新产生的 **Authentication**。

~~~java
protected InterceptorStatusToken beforeInvocation(Object object) {
    ......
​
        // Attempt to run as a different user
        Authentication runAs = this.runAsManager.buildRunAs(authenticated, object,
                                                            attributes);
​
    if (runAs == null) {
        if (debug) {
            logger.debug("RunAsManager did not change Authentication object");
        }
​
        // no further work post-invocation
        return new InterceptorStatusToken(SecurityContextHolder.getContext(), false,
                                          attributes, object);
    }
    else {
        if (debug) {
            logger.debug("Switching to RunAs Authentication: " + runAs);
        }
​
        SecurityContext origCtx = SecurityContextHolder.getContext();
        SecurityContextHolder.setContext(SecurityContextHolder.createEmptyContext());
        SecurityContextHolder.getContext().setAuthentication(runAs);
​
        // need to revert to token.Authenticated post-invocation
        return new InterceptorStatusToken(origCtx, true, attributes, object);
    }
}
~~~

注意，基类**AbstractSecurityInterceptor** 默认持有的是 RunAsManager 的空实现 NullRunAsManager。

~~~java
public abstract class AbstractSecurityInterceptor implements InitializingBean,
    ApplicationEventPublisherAware, MessageSourceAware {
  ......
  private RunAsManager runAsManager = new NullRunAsManager();
    ......
 }
~~~

待请求完成后会在 **finallyInvocation()** 中将原来的 **SecurityContext** 重新设置给**SecurityContextHolder**。

~~~java
protected void finallyInvocation(InterceptorStatusToken token) {
    if (token != null && token.isContextHolderRefreshRequired()) {
        if (logger.isDebugEnabled()) {
            logger.debug("Reverting to original Authentication: "
                         + token.getSecurityContext().getAuthentication());
        }
​
        SecurityContextHolder.setContext(token.getSecurityContext());
    }
}
~~~

**注意： 无论正常调用，亦或是请求异常等，都会触发 finallyInvocation()。**

~~~java
public void invoke(FilterInvocation fi) throws IOException, ServletException {
    if ((fi.getRequest() != null)
       ......
    }
    else {
        ......
​
        try {
            fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
        }
        finally {
            // 无论是否成功、抛异常与否，均会执行
            super.finallyInvocation(token);
        }
​
        // 正常请求结束，最后也会执行（afterInvocation 内部会调用finallyInvocation ）
        super.afterInvocation(token, null);
    }
}
~~~

即便是正常执行结束，依然会执行 **finallyInvocation()**（**afterInvocation** 内部会调用**finallyInvocation** ）。

~~~java
protected Object afterInvocation(InterceptorStatusToken token, Object returnedObject) {
    ......
​
    finallyInvocation(token); // continue to clean in this method for passivity
​
    ......
}
~~~

**AccessDecisionManager 是在访问受保护的对象之前判断用户是否拥有该对象的****访问****权限**。然而，有时候我们可能会希望在请求执行完成后对返回值做一些修改或者权限校验，当然，也可以简单的通过AOP来实现这一功能。

同样的，Spring Security 提供了 **AfterInvocationManager** 接口，它允许我们在受保护对象访问完成后对返回值进行修改或者进行权限校验，权限校验不通过时抛出 AccessDeniedException，并使用观察者模式通知其它Bean。

~~~java
protected Object afterInvocation(InterceptorStatusToken token, Object returnedObject) {
    ......
​
    if (afterInvocationManager != null) {
        ......
        catch (AccessDeniedException accessDeniedException) {
            AuthorizationFailureEvent event = new AuthorizationFailureEvent(
                token.getSecureObject(), token.getAttributes(), token
                .getSecurityContext().getAuthentication(),
                accessDeniedException);
            publishEvent(event);
​
            throw accessDeniedException;
        }
    }
  ......  
}
~~~

其将由 AbstractSecurityInterceptor 的子类进行调用，如默认子类 **FilterSecurityInterceptor** 。

需要特别注意的是，**AfterInvocationManager** 需要在受保护对象成功被访问后才能执行。

**Spring Security** 官方文档提供的 **AfterInvocationManager** 构造图如下：

![img](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210925152247.png)

类似于**AuthenticationManager**，**AfterInvocationManager** 同样也有一个默认的实现类AfterInvocationProviderManager，其中有一个由 AfterInvocationProvider 组成的集合属性。

~~~java
public class AfterInvocationProviderManager implements AfterInvocationManager,
    InitializingBean {
  ......
​
  private List<AfterInvocationProvider> providers;
​
    ......
}
~~~

**AfterInvocationProvider** 与 **AfterInvocationManager** 具有相同的方法定义。此一来，在调用**AfterInvocationProviderManager** 中的方法时，实际上就是依次调用其中成员属性 **providers** 中的**AfterInvocationProvider**  接口对应的方法。

~~~java
public Object decide(Authentication authentication, Object object,
                     Collection<ConfigAttribute> config, Object returnedObject)
    throws AccessDeniedException {
    Object result = returnedObject;
    for (AfterInvocationProvider provider : providers) {
        result = provider.decide(authentication, object, config, result);
    }
    return result;
}
~~~

而 **AfterInvocationProvider** 的默认实现类 **PostInvocationAdviceProvider** 中的 **PostInvocationAuthorizationAdvice**，其默认实现类 **ExpressionBasedPostInvocationAdvice**，正对应着后置权限注解 **@PostAuthorize** 

关于 **FILTER_APPLIED** 常量，在 **FilterSecurityInterceptor** 中是这么使用的：

~~~java
public void invoke(FilterInvocation fi) throws IOException, ServletException {
    if ((fi.getRequest() != null)
        && (fi.getRequest().getAttribute(FILTER_APPLIED) != null)
        && observeOncePerRequest) {
        // filter already applied to this request and user wants us to observe
        // once-per-request handling, so don't re-do security checking
        fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
    }
    else {
        // first time this request being called, so perform security checking
        if (fi.getRequest() != null && observeOncePerRequest) {
            fi.getRequest().setAttribute(FILTER_APPLIED, Boolean.TRUE);
        }
​
        ......
    }
}
~~~

其主要作用，是用于阻止请求的重复安全检查。

原理也简单，**第一次执行时，检查 request 中 FILTER_APPLIED 属性值为空，则放入值；后续该 request 再次请求时，FILTER_APPLIED 属性值不为空，代表已经进行过安全检查，则该请求直接通过，不再重复进行安全检查**。

### 12.2 FilterSecurityInterceptor默认初始化逻辑剖析

#### 12.2.1 SecurityMetadataSource

**Spring Security** 主要通过如下方法来配置默认的 **FilterSecurityInterceptor** 实例的 **SecurityMetadataSource**。

~~~java
http
  .authorizeRequests()
    .anyRequest()
    .authenticated()
    .antMatchers().permitAll()
    .antMatchers().hasRole()
    .antMatchers().hasAuthority()
  ......
~~~

#### 12.2.2 初始化逻辑

1. **AbstractInterceptUrlConfigurer**

在 **AbstractInterceptUrlConfigurer** 类的 configure 方法中定义实例。

~~~java
@Override
public void configure(H http) throws Exception {
    FilterInvocationSecurityMetadataSource metadataSource = createMetadataSource(http);
    if (metadataSource == null) {
        return;
    }
    FilterSecurityInterceptor securityInterceptor = createFilterSecurityInterceptor(
        http, metadataSource, http.getSharedObject(AuthenticationManager.class));
    if (filterSecurityInterceptorOncePerRequest != null) {
        securityInterceptor
            .setObserveOncePerRequest(filterSecurityInterceptorOncePerRequest);
    }
    securityInterceptor = postProcess(securityInterceptor);
    http.addFilter(securityInterceptor);
    http.setSharedObject(FilterSecurityInterceptor.class, securityInterceptor);
}
~~~

同样的，该实例也会放到 sharedObject 中。

**FilterSecurityInterceptor** 实例的创建是调用的 **AbstractInterceptUrlConfigurer** 类的 **createFilterSecurityInterceptor** 方法，创建逻辑如下：

~~~java
private FilterSecurityInterceptor createFilterSecurityInterceptor(H http,
      FilterInvocationSecurityMetadataSource metadataSource,
      AuthenticationManager authenticationManager) throws Exception {
    FilterSecurityInterceptor securityInterceptor = new FilterSecurityInterceptor();
    securityInterceptor.setSecurityMetadataSource(metadataSource);
    securityInterceptor.setAccessDecisionManager(getAccessDecisionManager(http));
    securityInterceptor.setAuthenticationManager(authenticationManager);
    securityInterceptor.afterPropertiesSet();
    return securityInterceptor;
}
~~~

**SecurityMetadataSource** 是通过 **AbstractInterceptUrlConfigurer** 类的抽象 createMetadataSource 方法来创建。

~~~java
/**
 * Subclasses should implement this method to provide a
 * {@link FilterInvocationSecurityMetadataSource} for the
 * {@link FilterSecurityInterceptor}.
 *
 * @param http the builder to use
 *
 * @return the {@link FilterInvocationSecurityMetadataSource} to set on the
 * {@link FilterSecurityInterceptor}. Cannot be null.
 */
abstract FilterInvocationSecurityMetadataSource createMetadataSource(H http);
~~~

其具体逻辑，是由 AbstractInterceptUrlConfigurer 类的子类 ExpressionUrlAuthorizationConfigurer 提供。为什么是这个类呢？看看最前面我们说的如何配置 FilterSecurityInterceptor 实例的 SecurityMetadataSource，就能基本明白。

~~~java
@Override
final ExpressionBasedFilterInvocationSecurityMetadataSource createMetadataSource(
    H http) {
    LinkedHashMap<RequestMatcher, Collection<ConfigAttribute>> requestMap = REGISTRY
        .createRequestMap();
    if (requestMap.isEmpty()) {
        throw new IllegalStateException(
            "At least one mapping is required (i.e. authorizeRequests().anyRequest().authenticated())");
    }
    return new ExpressionBasedFilterInvocationSecurityMetadataSource(requestMap,
                                                                     getExpressionHandler(http));
}
~~~

REGISTRY 是什么呢？正是 ExpressionInterceptUrlRegistry。可以看看 http.authorizeRequests() 返回值类型，正是 ExpressionInterceptUrlRegistry。而该类，正是 ExpressionUrlAuthorizationConfigurer 类的子类。

~~~java
public class ExpressionInterceptUrlRegistry
      extends
      ExpressionUrlAuthorizationConfigurer<H>.AbstractInterceptUrlRegistry<ExpressionInterceptUrlRegistry, AuthorizedUrl>
~~~

那再来看一下 requestMap 是如何创建的。requestMap 是由抽象类 AbstractConfigAttributeRequestMatcherRegistry 创建的。这个抽象类是 AbstractInterceptUrlRegistry 类的基类，而 AbstractInterceptUrlRegistry 类，正是 ExpressionInterceptUrlRegistry 类的基类。

~~~java
final LinkedHashMap<RequestMatcher, Collection<ConfigAttribute>> createRequestMap() {
    if (unmappedMatchers != null) {
        throw new IllegalStateException(
            "An incomplete mapping was found for "
            + unmappedMatchers
            + ". Try completing it with something like requestUrls().<something>.hasRole('USER')");
    }
    LinkedHashMap<RequestMatcher, Collection<ConfigAttribute>> requestMap = new LinkedHashMap<RequestMatcher, Collection<ConfigAttribute>>();
    for (UrlMapping mapping : getUrlMappings()) {
        RequestMatcher matcher = mapping.getRequestMatcher();
        Collection<ConfigAttribute> configAttrs = mapping.getConfigAttrs();
        requestMap.put(matcher, configAttrs);
    }
    return requestMap;
}
~~~

正是把前面 http 配置的 authorizeRequests，转化为 UrlMappings，然后再转换为 LinkedHashMap<RequestMatcher, Collection<ConfigAttribute>>。

在ExpressionUrlAuthorizationConfigurer类中

![image-20210925191153273](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210925191153.png)

而 **ExpressionUrlAuthorizationConfigurer** 类的 **interceptUrl()**方法，正是向 **UrlMappings** 中添加内容。

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210925191607.png" alt="image-20210925191607341" style="zoom:50%;" />

**SecurityMetadataSource** 的初始化基本完成。

2. **AccessDecisionManager**

认证管理器

默认的 AccessDecisionManager 初始化也是由 **AbstractInterceptUrlConfigurer** 类创建的。

~~~java
/**
 * Creates the default {@code AccessDecisionManager}
 * @return the default {@code AccessDecisionManager}
 */
private AccessDecisionManager createDefaultAccessDecisionManager(H http) {
    AffirmativeBased result = new AffirmativeBased(getDecisionVoters(http));
    return postProcess(result);
}
/**
 * If currently null, creates a default {@link AccessDecisionManager} using
 * {@link #createDefaultAccessDecisionManager(HttpSecurityBuilder)}. Otherwise returns the
 * {@link AccessDecisionManager}.
 *
 * @param http the builder to use
 *
 * @return the {@link AccessDecisionManager} to use
 */
private AccessDecisionManager getAccessDecisionManager(H http) {
    if (accessDecisionManager == null) {
        accessDecisionManager = createDefaultAccessDecisionManager(http);
    }
    return accessDecisionManager;
}
~~~

如果没有设置自定义的 accessDecisionManager，则会创建默认的 **AffirmativeBased** 实例。

3. **AccessDecisionVoter**

**AccessDecisionVoters** 也是由**AbstractInterceptUrlConfigurer** 类的抽象方法 getDecisionVoters 提供。

~~~Java
/**
 * Subclasses should implement this method to provide the {@link AccessDecisionVoter}
 * instances used to create the default {@link AccessDecisionManager}
 *
 * @param http the builder to use
 *
 * @return the {@link AccessDecisionVoter} instances used to create the default
 * {@link AccessDecisionManager}
 */
abstract List<AccessDecisionVoter<? extends Object>> getDecisionVoters(H http);
~~~

而真正的逻辑实现，是由其子类 **ExpressionUrlAuthorizationConfigurer** 提供。

~~~java
@Override
@SuppressWarnings("rawtypes")
final List<AccessDecisionVoter<? extends Object>> getDecisionVoters(H http) {
    List<AccessDecisionVoter<? extends Object>> decisionVoters = new ArrayList<AccessDecisionVoter<? extends Object>>();
    WebExpressionVoter expressionVoter = new WebExpressionVoter();
    expressionVoter.setExpressionHandler(getExpressionHandler(http));
    decisionVoters.add(expressionVoter);
    return decisionVoters;
}
~~~

4. **ExpressionHandler**

**ExpressionHandler** 的初始化逻辑是由 **ExpressionUrlAuthorizationConfigurer** 类的 **getExpressionHandler** 方法实现。

~~~java
private SecurityExpressionHandler<FilterInvocation> getExpressionHandler(H http) {
    if (expressionHandler == null) {
        DefaultWebSecurityExpressionHandler defaultHandler = new DefaultWebSecurityExpressionHandler();
        AuthenticationTrustResolver trustResolver = http
            .getSharedObject(AuthenticationTrustResolver.class);
        if (trustResolver != null) {
            defaultHandler.setTrustResolver(trustResolver);
        }
        ApplicationContext context = http.getSharedObject(ApplicationContext.class);
        if (context != null) {
            String[] roleHiearchyBeanNames = context.getBeanNamesForType(RoleHierarchy.class);
            if (roleHiearchyBeanNames.length == 1) {
                defaultHandler.setRoleHierarchy(context.getBean(roleHiearchyBeanNames[0], RoleHierarchy.class));
            }
            String[] grantedAuthorityDefaultsBeanNames = context.getBeanNamesForType(GrantedAuthorityDefaults.class);
            if (grantedAuthorityDefaultsBeanNames.length == 1) {
                GrantedAuthorityDefaults grantedAuthorityDefaults = context.getBean(grantedAuthorityDefaultsBeanNames[0], GrantedAuthorityDefaults.class);
                defaultHandler.setDefaultRolePrefix(grantedAuthorityDefaults.getRolePrefix());
            }
            String[] permissionEvaluatorBeanNames = context.getBeanNamesForType(PermissionEvaluator.class);
            if (permissionEvaluatorBeanNames.length == 1) {
                PermissionEvaluator permissionEvaluator = context.getBean(permissionEvaluatorBeanNames[0], PermissionEvaluator.class);
                defaultHandler.setPermissionEvaluator(permissionEvaluator);
            }
        }
        expressionHandler = postProcess(defaultHandler);
    }
    return expressionHandler;
}
~~~

首先，expressionHandler 是 DefaultWebSecurityExpressionHandler 实例。

如果存在 AuthenticationTrustResolver 实例，则设置到 DefaultWebSecurityExpressionHandler 实例中。

如果存在 RoleHierarchy 实例（隶属关系角色），同样设置到 DefaultWebSecurityExpressionHandler 实例中。

如果存在 GrantedAuthorityDefaults 实例（设置角色前缀的类），则设置该实例中定义的角色前缀到 DefaultWebSecurityExpressionHandler 实例中。

如果存在 PermissionEvaluator 实例，同样设置到 DefaultWebSecurityExpressionHandler 实例中。

PermissionEvaluator 用于确定用户是否具有权限或给定域对象的权限。


### 12.3 AccessDecisionVoter

一个接口，**AccessDecisionVoter** 是一个投票器，负责对授权决策进行表决。然后，最终由**AccessDecisionManager** 统计所有的投票器表决后，来做最终的授权决策。

几种常见的投票器：

1. WebExpressionVoter

最常用的，也是 Spring Security 框架默认 FilterSecurityInterceptor 实例中 AccessDecisionManager 默认的投票器 WebExpressionVoter。其实，就是对使用 http.authorizeRequests() 基于 Spring-EL进行控制权限的的授权决策类。

~~~java
http
    .authorizeRequests()
    .anyRequest()
    .authenticated()
    .antMatchers().permitAll()
    .antMatchers().hasRole()
    .antMatchers().hasAuthority()
    ......
~~~

2. AuthenticatedVoter

3. PreInvocationAuthorizationAdviceVoter

用于处理基于注解 **@PreFilter** 和 **@PreAuthorize** 生成的 **PreInvocationAuthorizationAdvice**，来处理授权决策的实现。

~~~java
public int vote(Authentication authentication, MethodInvocation method,
                Collection<ConfigAttribute> attributes) {
​
    // Find prefilter and preauth (or combined) attributes
    // if both null, abstain
    // else call advice with them
​
    PreInvocationAttribute preAttr = findPreInvocationAttribute(attributes);
​
    if (preAttr == null) {
        // No expression based metadata, so abstain
        return ACCESS_ABSTAIN;
    }
​
    boolean allowed = preAdvice.before(authentication, method, preAttr);
​
    return allowed ? ACCESS_GRANTED : ACCESS_DENIED;
}
~~~

4. RoleVoter

角色投票器。用于 ConfigAttribute#getAttribute() 中配置为角色的授权决策。其默认前缀为 ROLE_，可以自定义，也可以设置为空，直接使用角色标识进行判断。

~~~java
public int vote(Authentication authentication, Object object,
      Collection<ConfigAttribute> attributes) {
    if (authentication == null) {
        return ACCESS_DENIED;
    }
    int result = ACCESS_ABSTAIN;
    Collection<? extends GrantedAuthority> authorities = extractAuthorities(authentication);
​
    for (ConfigAttribute attribute : attributes) {
        if (this.supports(attribute)) {
            result = ACCESS_DENIED;
​
            // Attempt to find a matching granted authority
            for (GrantedAuthority authority : authorities) {
                if (attribute.getAttribute().equals(authority.getAuthority())) {
                    return ACCESS_GRANTED;
                }
            }
        }
    }
​
    return result;
}
~~~

**注意，决策策略比较简单，用户只需拥有任一当前请求需要的角色即可，不必全部拥有**。

### 12.3 **AccessDecisionManager**

决策管理器

做出最终的访问控制（授权）决定。

常用的AccessDecisionManager有三个：

1. **AffirmativeBased**

**Spring Security** 框架默认的 **AccessDecisionManager。**

~~~java
/**
 * Simple concrete implementation of
 * {@link org.springframework.security.access.AccessDecisionManager} that grants access if
 * any <code>AccessDecisionVoter</code> returns an affirmative response.
 */
~~~

**只要任一 AccessDecisionVoter 返回肯定的结果，便授予访问权限**。

其决策逻辑：

~~~java
public void decide(Authentication authentication, Object object,
      Collection<ConfigAttribute> configAttributes) throws AccessDeniedException {
    int deny = 0;
​
    for (AccessDecisionVoter voter : getDecisionVoters()) {
        int result = voter.vote(authentication, object, configAttributes);
​
        if (logger.isDebugEnabled()) {
            logger.debug("Voter: " + voter + ", returned: " + result);
        }
​
        switch (result) {
            case AccessDecisionVoter.ACCESS_GRANTED:
                return;
​
            case AccessDecisionVoter.ACCESS_DENIED:
                deny++;
​
                break;
​
            default:
                break;
        }
    }
​
    if (deny > 0) {
        throw new AccessDeniedException(messages.getMessage(
            "AbstractAccessDecisionManager.accessDenied", "Access is denied"));
    }
​
    // To get this far, every AccessDecisionVoter abstained
    checkAllowIfAllAbstainDecisions();
}
~~~



所有当前请求需要的 ConfigAttributes ，全部交给 AccessDecisionVoter 进行投票。只要任一 AccessDecisionVoter 授予访问权限，便返回，不再继续决策判断。否则，便抛出访问授权拒绝的异常，即：AccessDeniedException。

最后是根据配置，判断如果全部 **AccessDecisionVoter** 都授权决策中立时的授权决策。

2. **ConsensusBased**

**少数服从多数授权访问决策方案**。

~~~java
public void decide(Authentication authentication, Object object,
      Collection<ConfigAttribute> configAttributes) throws AccessDeniedException {
    int grant = 0;
    int deny = 0;
​
    for (AccessDecisionVoter voter : getDecisionVoters()) {
        int result = voter.vote(authentication, object, configAttributes);
​
        if (logger.isDebugEnabled()) {
            logger.debug("Voter: " + voter + ", returned: " + result);
        }
​
        switch (result) {
            case AccessDecisionVoter.ACCESS_GRANTED:
                grant++;
​
                break;
​
            case AccessDecisionVoter.ACCESS_DENIED:
                deny++;
​
                break;
​
            default:
                break;
        }
    }
​
    if (grant > deny) {
        return;
    }
​
    if (deny > grant) {
        throw new AccessDeniedException(messages.getMessage(
            "AbstractAccessDecisionManager.accessDenied", "Access is denied"));
    }
​
    if ((grant == deny) && (grant != 0)) {
        if (this.allowIfEqualGrantedDeniedDecisions) {
            return;
        }
        else {
            throw new AccessDeniedException(messages.getMessage(
                "AbstractAccessDecisionManager.accessDenied", "Access is denied"));
        }
    }
​
    // To get this far, every AccessDecisionVoter abstained
    checkAllowIfAllAbstainDecisions();
}
~~~

少数服从多数判断逻辑。需要注意，就是授予权限和拒绝权限相等时的逻辑。其实，该决策器也考虑到了这一点，所以提供了 allowIfEqualGrantedDeniedDecisions 参数，用于给用户提供自定义的机会，其默认值为 true，即代表允许授予权限和拒绝权限相等，且同时也代表授予访问权限。

3. **UnanimousBased**

全部投票通过才认证成功。

最严格的的授权决策器。要求所有 **AccessDecisionVoter** 均返回肯定的结果时，才代表授予权限。

~~~Java
/**
 * Simple concrete implementation of
 * {@link org.springframework.security.access.AccessDecisionManager} that requires all
 * voters to abstain or grant access.
 */
~~~

其决策逻辑如下。

~~~java
public void decide(Authentication authentication, Object object,
      Collection<ConfigAttribute> attributes) throws AccessDeniedException {
​
    int grant = 0;
​
    List<ConfigAttribute> singleAttributeList = new ArrayList<>(1);
    singleAttributeList.add(null);
​
    for (ConfigAttribute attribute : attributes) {
        singleAttributeList.set(0, attribute);
​
        for (AccessDecisionVoter voter : getDecisionVoters()) {
            int result = voter.vote(authentication, object, singleAttributeList);
​
            if (logger.isDebugEnabled()) {
                logger.debug("Voter: " + voter + ", returned: " + result);
            }
​
            switch (result) {
                case AccessDecisionVoter.ACCESS_GRANTED:
                    grant++;
​
                    break;
​
                case AccessDecisionVoter.ACCESS_DENIED:
                    throw new AccessDeniedException(messages.getMessage(
                        "AbstractAccessDecisionManager.accessDenied",
                        "Access is denied"));
​
                default:
                    break;
            }
        }
    }
​
    // To get this far, there were no deny votes
    if (grant > 0) {
        return;
    }
​
    // To get this far, every AccessDecisionVoter abstained
    checkAllowIfAllAbstainDecisions();
}
~~~

可以看到，同前两个决策器不同之处在于，循环将每一个当前请求需要的 ConfigAttribute 传递给 AccessDecisionVoter 进行决策，而不是全部传递过去。这就代表每一个  ConfigAttribute 每一个 AccessDecisionVoter 均需返回肯定的结果才可以授予权限。所以，最为严格。

## 13. 自定义用户名密码参数名及用户名密码验证路径

无论是表单登录配置器（FormLoginConfigurer），还是用户密码验证Filter（UsernamePasswordAuthenticationFilter），**Spring Security** 默认的用户名、密码参数名均为 `username`、`password`。

表单登录配置：

~~~java
public FormLoginConfigurer() {
    super(new UsernamePasswordAuthenticationFilter(), null);
    usernameParameter("username");
    passwordParameter("password");
}
~~~

用户密码验证Filter：

~~~java
public class UsernamePasswordAuthenticationFilter extends
        AbstractAuthenticationProcessingFilter {
    // ~ Static fields/initializers
    // =====================================================================================
    public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";
    public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";
    private String usernameParameter = SPRING_SECURITY_FORM_USERNAME_KEY;
    private String passwordParameter = SPRING_SECURITY_FORM_PASSWORD_KEY;
~~~

同样的，还有用户密码验证Filter（UsernamePasswordAuthenticationFilter）的拦截路径，即用户名密码验证路径，**Spring Security** 默认为 `/login`，且只支持 `POST` 方式访问。

~~~java
public UsernamePasswordAuthenticationFilter() {
        super(new AntPathRequestMatcher("/login", "POST"));
    }
    ......
    public Authentication attemptAuthentication(HttpServletRequest request,
            HttpServletResponse response) throws AuthenticationException {
        if (postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException(
                    "Authentication method not supported: " + request.getMethod());
        }
        ......
    }
~~~

首先就是用户名密码验证路径，非常容易加一项配置即可。

~~~java
protected void configure(HttpSecurity http) throws Exception {
    http
        .formLogin()
        .loginProcessingUrl("/j_spring_security_check")
        ...... 
}
~~~

然后，就是修改用户名、密码参数名称。

~~~java
protected void configure(HttpSecurity http) throws Exception {
    http
        .formLogin()
        ......
        .usernameParameter("j_username")
        .passwordParameter("j_password")
        ......
}
~~~

## 14. UsernamePasswordAuthenticationFilter

在 **Spring Security** 框架中，最常用的 Filter 便是表单登录Filter，即 UsernamePasswordAuthenticationFilter。

![image-20211009104304379](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20211009104304.png)

从上图中，能清晰的了解到 UsernamePasswordAuthenticationFilter 的继承关系，也确实实现了 Filter 接口。

那么，我们便从 Filter 的核心方法 doFilter 开始看起，然后，内部贯穿说明其它涉及的逻辑。

基类`AbstractAuthenticationProcessingFilter`方法中重写了doFilter方法：

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {

		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;

		if (!requiresAuthentication(request, response)) {
			chain.doFilter(request, response);

			return;
		}
  、、、
	}
```

如不需要认证，则直接执行下一个 Filter；如果需要认证，再向下继续进行。按照 Spring Security 框架的默认配置，此处只拦截 /login 的action，也即表单登录提交action，不会拦截其它请求。因此，这也就是解释了为何其它请求不会被 UsernamePasswordAuthenticationFilter 处理。

此一节是通过 requiresAuthentication 方法来判断的。

```java
protected boolean requiresAuthentication(HttpServletRequest request,
      HttpServletResponse response) {
    return requiresAuthenticationRequestMatcher.matches(request);
}
```

而 requiresAuthenticationRequestMatcher，看着不熟悉，其实，我们一开始就为其赋了值。

比如构造方法（基类AbstractAuthenticationProcessingFilter）。

```java
protected AbstractAuthenticationProcessingFilter(
      RequestMatcher requiresAuthenticationRequestMatcher) {
    Assert.notNull(requiresAuthenticationRequestMatcher,
                   "requiresAuthenticationRequestMatcher cannot be null");
    this.requiresAuthenticationRequestMatcher = requiresAuthenticationRequestMatcher;
}
```

同时，UsernamePasswordAuthenticationFilter 默认的无参构造方法，也对此进行了赋值（Ant风格路径匹配）。

```java
public UsernamePasswordAuthenticationFilter() {
    super(new AntPathRequestMatcher("/login", "POST"));
}
```

**注意，正是此处，默认了 `/login`（老版本为** `/j_spring_security_check`）为用户名密码验证路径，即 `loginProcessingUrl`（老版本为 `filterProcessingUrl`）。

基类 AbstractAuthenticationProcessingFilter 的无参构造方法为：

```java
protected AbstractAuthenticationProcessingFilter(String defaultFilterProcessesUrl) {
    setFilterProcessesUrl(defaultFilterProcessesUrl);
}
```

基类的 setFilterProcessesUrl 方法和 requiresAuthenticationRequestMatcher 的 setter 方法一起，实现了对 requiresAuthenticationRequestMatcher 的赋值。

```java
public void setFilterProcessesUrl(String filterProcessesUrl) {
    setRequiresAuthenticationRequestMatcher(new AntPathRequestMatcher(
        filterProcessesUrl));
}
​
public final void setRequiresAuthenticationRequestMatcher(
    RequestMatcher requestMatcher) {
    Assert.notNull(requestMatcher, "requestMatcher cannot be null");
    this.requiresAuthenticationRequestMatcher = requestMatcher;
}
```

同时，表单登录配置器同时也默认了 `UsernamePasswordAuthenticationFilter` 作为表单登录验证的Filter，同时，提供了相关API以供相关配置属性的修改。

```java
/**
 * Creates a new instance
 * @see HttpSecurity#formLogin()
 */
public FormLoginConfigurer() {
    super(new UsernamePasswordAuthenticationFilter(), null);
    usernameParameter("username");
    passwordParameter("password");
}
​
......
​
@Override
public FormLoginConfigurer<H> loginPage(String loginPage) {
    return super.loginPage(loginPage);
}
​
......
​
public FormLoginConfigurer<H> usernameParameter(String usernameParameter) {
    getAuthenticationFilter().setUsernameParameter(usernameParameter);
    return this;
}
​
......
```

**尝试认证操作**

如果需要进行认证，也即就是表单提交操作，此时，便需要进行用户名、密码验证。

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
      throws IOException, ServletException {
​
    ......
​
    try {
      authResult = attemptAuthentication(request, response);
      ......
    }
```

具体的验证逻辑，在 UsernamePasswordAuthenticationFilter 中的 attemptAuthentication方法。

```java
public Authentication attemptAuthentication(HttpServletRequest request,
      HttpServletResponse response) throws AuthenticationException {
    if (postOnly && !request.getMethod().equals("POST")) {
        throw new AuthenticationServiceException(
            "Authentication method not supported: " + request.getMethod());
    }
​
    String username = obtainUsername(request);
    String password = obtainPassword(request);
​
    if (username == null) {
        username = "";
    }
​
    if (password == null) {
        password = "";
    }
​
    username = username.trim();
​
    UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
        username, password);
​
    // Allow subclasses to set the "details" property
    setDetails(request, authRequest);
​
    return this.getAuthenticationManager().authenticate(authRequest);
}
```

此处的验证逻辑比较简单，**最主要的便是调用 AuthenticationManager 进行身份认证**。

attemptAuthentication 方法主要是搜集参数（username、password），封装为验证请求（UsernamePasswordAuthenticationToken），然后调用AuthenticationManager 进行身份认证。关于认证的详细逻辑，后续会专门细讲，此处暂且不提。

**Session策略**

认证完成后（不发生异常），会有一个 Session 策略处理器处理 Session 的过程。

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
      throws IOException, ServletException {
​
    ......
​
    try {
      authResult = attemptAuthentication(request, response);
      if (authResult == null) {
        // return immediately as subclass has indicated that it hasn't completed
        // authentication
        return;
      }
      sessionStrategy.onAuthentication(authResult, request, response);
    }
```

**认证成功**

如果认证过程中没有发生异常，那么即为认证成功。

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
      throws IOException, ServletException {
​
    ......
​
    successfulAuthentication(request, response, chain, authResult);
  }
```

具体的认证成功逻辑如下：

```java
protected void successfulAuthentication(HttpServletRequest request,
      HttpServletResponse response, FilterChain chain, Authentication authResult)
      throws IOException, ServletException {
​
    if (logger.isDebugEnabled()) {
        logger.debug("Authentication success. Updating SecurityContextHolder to contain: "
                     + authResult);
    }
​
    SecurityContextHolder.getContext().setAuthentication(authResult);
​
    rememberMeServices.loginSuccess(request, response, authResult);
​
    // Fire event
    if (this.eventPublisher != null) {
        eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(
            authResult, this.getClass()));
    }
​
    successHandler.onAuthenticationSuccess(request, response, authResult);
}
```

暂且先不管 rememberMeServices。单看其它逻辑，也不复杂。首先把认证后的结果（UsernamePasswordAuthenticationToken ）放到了 SecurityContextHolder 属于当前线程的 SecurityContext 中；然后，使用 rememberMeServices 处理记住我逻辑；接着，如果事件发布器不为空，则发布认证成功消息，供其他 Bean 接收并处理；最后，调用 AuthenticationSuccessHandler 进行认证成功的相关操作。

> 此处简单说明一下，为何把**认证后的结果（****UsernamePasswordAuthenticationToken** **）放到了 SecurityContextHolder 属于当前线程的 SecurityContext 中。**

一切皆因 **SecurityContextHolder** 的初始化方法和 `strategyName` 参数。

```java
private static void initialize() {
    if (!StringUtils.hasText(strategyName)) {
        // Set default
        strategyName = MODE_THREADLOCAL;
    }
​
    if (strategyName.equals(MODE_THREADLOCAL)) {
        strategy = new ThreadLocalSecurityContextHolderStrategy();
    }
    else if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {
        strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
    }
    else if (strategyName.equals(MODE_GLOBAL)) {
        strategy = new GlobalSecurityContextHolderStrategy();
    }
    else {
        // Try to load a custom strategy
        try {
            Class<?> clazz = Class.forName(strategyName);
            Constructor<?> customStrategy = clazz.getConstructor();
            strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();
        }
        catch (Exception ex) {
            ReflectionUtils.handleReflectionException(ex);
        }
    }
​
    initializeCount++;
}
```

由于 `strategyName` 参数默认为空，所以，`SecurityContextHolderStrategy` 便被初始化为 `ThreadLocalSecurityContextHolderStrategy`。当然，也可以设置其它类型的 `ThreadLocalSecurityContextHolderStrategy`，只需设置 `strategyName` 参数即可。

**认证失败**

如果认证过程中发生异常，即代表认证失败。

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
      throws IOException, ServletException {
​
    ......
​
    catch (InternalAuthenticationServiceException failed) {
      logger.error(
          "An internal error occurred while trying to authenticate the user.",
          failed);
      unsuccessfulAuthentication(request, response, failed);
​
      return;
    }
    catch (AuthenticationException failed) {
      // Authentication failed
      unsuccessfulAuthentication(request, response, failed);
​
      return;
    }
​
    ......
  }
```

认证过程中发生异常，一共有分为两种情况，一是不好区分具体的认证失败原因、场景的内部异常，即 InternalAuthenticationServiceException；另一种就是父异常 AuthenticationException。这个异常是所有认证异常的父类。除了内部异常打印了一句日志，其它也没有明显的区别。

认证失败的具体处理逻辑如下：

```java
protected void unsuccessfulAuthentication(HttpServletRequest request,
      HttpServletResponse response, AuthenticationException failed)
      throws IOException, ServletException {
    SecurityContextHolder.clearContext();
​
    if (logger.isDebugEnabled()) {
        logger.debug("Authentication request failed: " + failed.toString(), failed);
        logger.debug("Updated SecurityContextHolder to contain null Authentication");
        logger.debug("Delegating to authentication failure handler " + failureHandler);
    }
​
    rememberMeServices.loginFail(request, response);
​
    failureHandler.onAuthenticationFailure(request, response, failed);
}
```

首先，清除放到了 SecurityContextHolder 属于当前线程的 SecurityContext 中的认证后的结果（UsernamePasswordAuthenticationToken ），然后，就是 rememberMeServices 的处理逻辑（暂时不谈）；最后，便是调用认证失败handler，即 AuthenticationFailureHandler 来处理失败后的逻辑。


## 15. **DaoAuthenticationProvider**

**AuthenticationProvider接口**

主要用于解析特定 `Authentication` 。

~~~Java
/**
 * Indicates a class can process a specific
 * {@link org.springframework.security.core.Authentication} implementation.
 */
~~~

有两个接口方法。

`authenticate(Authentication authentication)` 方法。

~~~java
Authentication authenticate(Authentication authentication)
    throws AuthenticationException;
~~~

返回包含凭据的完整身份验证对象 `authentication`。

 `supports(Class<?> authentication)` 方法。

~~~java
boolean supports(Class<?> authentication);
~~~

如果 AuthenticationProvider 支持给定的 Authentication 的话，会返回 true。但是，这并不保证 AuthenticationProvider 能够对给定的 Authentication 进行身份认证，它只是表明它可以支持对其进行更深入的评估，AuthenticationProvider 依然可以返回 null，以指示应尝试另一个 AuthenticationProvider。
此方法是用以选择一个能够匹配 `Authentication` 以胜任身份认证工作的 `AuthenticationProvider`，交给`ProviderManager` 来执行。

**AbstractUserDetailsAuthenticationProvider**

用于解析 `UsernamePasswordAuthenticationToken` 以进行身份认证的基础 `AuthenticationProvider`。其子类可以重写或使用其 `UserDetails` 对象。

其实现了 `AuthenticationProvider` 接口的 `authenticate(Authentication authentication)` 方法，其认证过程可分为一下几个步骤。

1. 获取用户名。

~~~java
String username = (authentication.getPrincipal() == null) ? "NONE_PROVIDED"
        : authentication.getName();
~~~



2. 缓存中获取 User。

~~~java
boolean cacheWasUsed = true;
    UserDetails user = this.userCache.getUserFromCache(username);
~~~



3. 如果缓存中获取的 User 为 null，再调用 **retrieveUser**（子类实现）方法获取。

~~~java
if (user == null) {
    cacheWasUsed = false;
​
    try {
        user = retrieveUser(username,
                            (UsernamePasswordAuthenticationToken) authentication);
    }
    catch (UsernameNotFoundException notFound) {
        logger.debug("User '" + username + "' not found");
​
        if (hideUserNotFoundExceptions) {
            throw new BadCredentialsException(messages.getMessage(
                "AbstractUserDetailsAuthenticationProvider.badCredentials",
                "Bad credentials"));
        }
        else {
            throw notFound;
        }
    }
​
    ......
}
~~~

如果找不到用户，会抛出 UsernameNotFoundException 异常。此时，处理方案有2个。

- 如果 hideUserNotFoundExceptions 为 true（默认为 true），即隐藏用户未找到异常，则会重新抛出凭据/密码错误异常，异常信息为 Spring Security 框架已定义好的提示信息。

- 如果不隐藏用户未找到异常，则直接抛出 UsernameNotFoundException 异常。

4. 前置身份认证检查。

~~~Java
preAuthenticationChecks.check(user);
~~~

默认的前置身份认证检查逻辑如下，**即事先校验一下用户的账号是否锁定、是否可用、是否过期**。

~~~java
private class DefaultPreAuthenticationChecks implements UserDetailsChecker {
    public void check(UserDetails user) {
        if (!user.isAccountNonLocked()) {
            logger.debug("User account is locked");
​
            throw new LockedException(messages.getMessage(
                "AbstractUserDetailsAuthenticationProvider.locked",
                "User account is locked"));
        }
​
        if (!user.isEnabled()) {
            logger.debug("User account is disabled");
​
            throw new DisabledException(messages.getMessage(
                "AbstractUserDetailsAuthenticationProvider.disabled",
                "User is disabled"));
        }
​
        if (!user.isAccountNonExpired()) {
            logger.debug("User account is expired");
​
            throw new AccountExpiredException(messages.getMessage(
                "AbstractUserDetailsAuthenticationProvider.expired",
                "User account has expired"));
        }
    }
}
~~~

5. 额外身份认证校验，对于 `UsernamePasswordAuthenticationToken` 来说就是凭据/密码校验。

~~~java
try {
    preAuthenticationChecks.check(user);
    additionalAuthenticationChecks(user,
           (UsernamePasswordAuthenticationToken) authentication);
}
catch (AuthenticationException exception) {
    if (cacheWasUsed) {
        // There was a problem, so try again after checking
        // we're using latest data (i.e. not from the cache)
        cacheWasUsed = false;
        user = retrieveUser(username,
                            (UsernamePasswordAuthenticationToken) authentication);
        preAuthenticationChecks.check(user);
        additionalAuthenticationChecks(user,
                                       (UsernamePasswordAuthenticationToken) authentication);
    }
    else {
        throw exception;
    }
}
~~~

如果认证过程中发生异常，会有如下处理逻辑：

- 如果当前的用户是从用户缓存中取出的，则使用原有的用户信息再进行一次身份认证，即获取用户信息、前置身份认证检查、额外身份认证检查；

- 如果不是从用户缓存中取出的，则直接抛出异常。

6. 后置身份认证检查。

~~~Java
postAuthenticationChecks.check(user);
~~~

默认的后置身份认证检查逻辑如下，即检查一下用户的凭据/密码是否过期。

~~~Java
private class DefaultPostAuthenticationChecks implements UserDetailsChecker {
    public void check(UserDetails user) {
        if (!user.isCredentialsNonExpired()) {
            logger.debug("User account credentials have expired");
​
            throw new CredentialsExpiredException(messages.getMessage(
                "AbstractUserDetailsAuthenticationProvider.credentialsExpired",
                "User credentials have expired"));
        }
    }
}
~~~

7. 将当前用户放入用户缓存，如果当前用户还没有被用户缓存缓存的话。

~~~java
if (!cacheWasUsed) {
    this.userCache.putUserInCache(user);
}
~~~

8. 转换 principal 为字符串类型。不过，需要 `forcePrincipalAsString` 参数为 `true`（默认为 `false`）。

~~~java
Object principalToReturn = user;
​
if (forcePrincipalAsString) {
    principalToReturn = user.getUsername();
}
~~~

9. 最后，创建身份认证成功的 `Authentication`。

~~~java
return createSuccessAuthentication(principalToReturn, authentication, user);
~~~

默认的创建身份认证成功的 `Authentication` 逻辑如下。

~~~Java
	protected Authentication createSuccessAuthentication(Object principal,
			Authentication authentication, UserDetails user) {
		// Ensure we return the original credentials the user supplied,
		// so subsequent attempts are successful even with encoded passwords.
		// Also ensure we return the original getDetails(), so that future
		// authentication events after cache expiry contain the details
		UsernamePasswordAuthenticationToken result = new UsernamePasswordAuthenticationToken(
				principal, authentication.getCredentials(),
				authoritiesMapper.mapAuthorities(user.getAuthorities()));
		result.setDetails(authentication.getDetails());

		return result;
	}
~~~

即创建了一个新的 UsernamePasswordAuthenticationToken，**与未认证的区别就是 principal 变成了检索到的用户详细信息（或者用户名，强制字符串principal）**。

当然，此方法是 protected 类型的，子类可以重写。因为，子类通常在 `Authentication` 中存储用户提供的原始凭据/密码，而非加盐、加密过的。

`AbstractUserDetailsAuthenticationProvider` 的主要功能就是这些，剩下的，就是诸如 `retrieveUser` 、`additionalAuthenticationChecks` 等需要子类实现的抽象方法了，这就是属于子类 `DaoAuthenticationProvider` 的部分了。

**DaoAuthenticationProvider**

说到 `Authentication` 相信都不陌生。最著名的，便是 `UsernamePasswordAuthenticationToken`。而 `DaoAuthenticationProvider` 便是用于解析并认证 `UsernamePasswordAuthenticationToken` 的这样一个认证服务提供者。

其最终目的，就是根据 `UsernamePasswordAuthenticationToken`，获取到 `username`，然后调用 `UserDetailsService` 检索用户详细信息。

在其基类 `AbstractUserDetailsAuthenticationProvider` 中，我们已经讲过，需要子类实现 `retrieveUser` 、`additionalAuthenticationChecks` 等抽象方法。

1. 额外的身份认证检查，也即 **additionalAuthenticationChecks**，密码检查。

```java
protected void additionalAuthenticationChecks(UserDetails userDetails,
      UsernamePasswordAuthenticationToken authentication)
      throws AuthenticationException {
    if (authentication.getCredentials() == null) {
        logger.debug("Authentication failed: no credentials provided");
​
        throw new BadCredentialsException(messages.getMessage(
            "AbstractUserDetailsAuthenticationProvider.badCredentials",
            "Bad credentials"));
    }
​
    String presentedPassword = authentication.getCredentials().toString();
​
    if (!passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
        logger.debug("Authentication failed: password does not match stored value");
​
        throw new BadCredentialsException(messages.getMessage(
            "AbstractUserDetailsAuthenticationProvider.badCredentials",
            "Bad credentials"));
    }
}
```

2. 接下来就是用户检索，即 **retrieveUser**。

```Java
protected final UserDetails retrieveUser(String username,
      UsernamePasswordAuthenticationToken authentication)
      throws AuthenticationException {
    prepareTimingAttackProtection();
    try {
        UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
        if (loadedUser == null) {
            throw new InternalAuthenticationServiceException(
                "UserDetailsService returned null, which is an interface contract violation");
        }
        return loadedUser;
    }
    catch (UsernameNotFoundException ex) {
        mitigateAgainstTimingAttack(authentication);
        throw ex;
    }
    catch (InternalAuthenticationServiceException ex) {
        throw ex;
    }
    catch (Exception ex) {
        throw new InternalAuthenticationServiceException(ex.getMessage(), ex);
    }
}
```

需要调用 UserDetailsService 检索用户详细信息，如权限列表、存储密码等，逻辑也比较简单。但是，有两个特殊方法需要注意一下，即检索用户前调用的 prepareTimingAttackProtection() 方法，和抛出 UsernameNotFoundException 异常后调用的 mitigateAgainstTimingAttack(authentication) 方法。

这两个方法都是用于**定时攻击保护**的。

首先是 `prepareTimingAttackProtection()` 方法。

```java
private void prepareTimingAttackProtection() {
    if (this.userNotFoundEncodedPassword == null) {
        this.userNotFoundEncodedPassword = this.passwordEncoder.encode(USER_NOT_FOUND_PASSWORD);
    }
}
```

该方法从方法命名上理解，是为了准备定时攻击保护用的。其实，就是将 userNotFoundEncodedPassword，也即用户未找到时的加密密码给准备好，然后使用配置的 passwordEncoder 来加密为密文，默认的用户未找到时的明文密码为 `userNotFoundPassword`。

```java
private static final String USER_NOT_FOUND_PASSWORD = "userNotFoundPassword";
```

其次是 `mitigateAgainstTimingAttack(authentication)` 方法。

```java
private void mitigateAgainstTimingAttack(UsernamePasswordAuthenticationToken authentication) {
    if (authentication.getCredentials() != null) {
        String presentedPassword = authentication.getCredentials().toString();
        this.passwordEncoder.matches(presentedPassword, this.userNotFoundEncodedPassword);
    }
}
```

逻辑也不复杂，就是在用户提供的凭据/密码不为空时，使用配置的 `passwordEncoder` 来验证两者是否相等。

这里我们看一下方法命名中的 mitigate，意味缓解、减轻、缓和。

> 整体流程：

> 用户登录时，调用 AuthenticationProvider 的 authenticate 方法，然后，先从用户缓存中获取用户，如果取不到，再调用 retrieveUser 方法检索用户。最后，再调用 additionalAuthenticationChecks 方法进行密码检查。

另外，**DaoAuthenticationProvider** 还重写了基类的 **createSuccessAuthentication** 方法。

```java
@Override
protected Authentication createSuccessAuthentication(Object principal,
                                                     Authentication authentication, UserDetails user) {
    boolean upgradeEncoding = this.userDetailsPasswordService != null
        && this.passwordEncoder.upgradeEncoding(user.getPassword());
    if (upgradeEncoding) {
        String presentedPassword = authentication.getCredentials().toString();
        String newPassword = this.passwordEncoder.encode(presentedPassword);
        user = this.userDetailsPasswordService.updatePassword(user, newPassword);
    }
    return super.createSuccessAuthentication(principal, authentication, user);
}
```

至于 `userDetailsPasswordService`，其实也很简单，就是更新一下 User 中的密码，仅此而已。

```java
public UserDetails updatePassword(UserDetails user, String newPassword) {
    String username = user.getUsername();
    MutableUserDetails mutableUser = this.users.get(username.toLowerCase());
    mutableUser.setPassword(newPassword);
    return mutableUser;
}
```

## 16. AuthenticationManager

Spring Security 框架中的另一个重要接口 `**AuthenticationManager**`， 被设计用于处理 `Authentication` 请求。

与 `AuthenticationProvider` 接口一致，`AuthenticationManager` 接口中有且只有一个方法，即`authenticate(Authentication authentication)` 方法。

```java
Authentication authenticate(Authentication authentication)
      throws AuthenticationException;
```

该方法与 AuthenticationProvider 中的 authenticate 方法声明及功能完全一致，返回包含凭据的完整身份验证对象 authentication。但是，如果 AuthenticationProvider 不支持给定的 Authentication 的话，该方法可能会返回 null。在此情况下，下一个支持 authentication 的 AuthenticationProvider 将会被尝试。
`AuthenticationManager` 接口的默认实现为 `ProviderManager`，其逻辑也不复杂。

首先，便是调用 `AuthenticationProvider` 中的 `supports(Class<?> authentication)` 方法，判断是否支持当前的 `Authentication` 请求。

```Java
public Authentication authenticate(Authentication authentication)
      throws AuthenticationException {
    ......
​
    for (AuthenticationProvider provider : getProviders()) {
      if (!provider.supports(toTest)) {
        continue;
      }
```

只有支持当前 `Authentication` 请求的 `AuthenticationProvider` 才会继续后续逻辑处理。

然后，便是调用 `AuthenticationProvider` 中的 `authenticate(Authentication authentication)` 方法进行身份认证。

```java
public Authentication authenticate(Authentication authentication)
      throws AuthenticationException {
    ......
​
    for (AuthenticationProvider provider : getProviders()) {
      if (!provider.supports(toTest)) {
        continue;
      }
​
      ......
​
      try {
        result = provider.authenticate(authentication);
        ......
      }
```

如果认证成功且返回的结果不为 `null`，则执行 authentication details 的拷贝逻辑*。*

```java
try {
    result = provider.authenticate(authentication);
​
    if (result != null) {
        copyDetails(authentication, result);
        break;
    }
}
​
......
​
private void copyDetails(Authentication source, Authentication dest) {
    if ((dest instanceof AbstractAuthenticationToken) && (dest.getDetails() == null)) {
        AbstractAuthenticationToken token = (AbstractAuthenticationToken) dest;
​
        token.setDetails(source.getDetails());
    }
}
```

如果发生 AccountStatusException 或 InternalAuthenticationServiceException 异常，则会通过Spring事件发布器`AuthenticationEventPublisher` 发布异常事件。

```java
catch (AccountStatusException e) {
    prepareException(e, authentication);
    // SEC-546: Avoid polling additional providers if auth failure is due to
    // invalid account status
    throw e;
}
catch (InternalAuthenticationServiceException e) {
    prepareException(e, authentication);
    throw e;
}
​
......
​
private void prepareException(AuthenticationException ex, Authentication auth) {
    eventPublisher.publishAuthenticationFailure(ex, auth);
}
```

如果异常为其它类型的 `AuthenticationException`，则将此异常设置为 `lastException` 并返回。

```java
catch (AuthenticationException e) {
    lastException = e;
}
```

如果认证结果为 `null`，且存在父 `AuthenticationManager`，则调用父 `AuthenticationManager` 进行同样的身份认证操作，其处理逻辑基本同上。

```java
if (result == null && parent != null) {
    // Allow the parent to try.
    try {
        result = parentResult = parent.authenticate(authentication);
    }
    catch (ProviderNotFoundException e) {
        // ignore as we will throw below if no other exception occurred prior to
        // calling parent and the parent
        // may throw ProviderNotFound even though a provider in the child already
        // handled the request
    }
    catch (AuthenticationException e) {
        lastException = parentException = e;
    }
}
```

如果认证结果不为 null，同时，此时的 eraseCredentialsAfterAuthentication 参数为 true，且此时认证后的 Authentication 实现了 CredentialsContainer 接口，那么即调用 CredentialsContainer 接口的凭据擦除方法，即eraseCredentials，擦除相关凭据信息。

```java
if (result != null) {
    if (eraseCredentialsAfterAuthentication
        && (result instanceof CredentialsContainer)) {
        // Authentication is complete. Remove credentials and other secret data
        // from authentication
        ((CredentialsContainer) result).eraseCredentials();
    }
​
    // If the parent AuthenticationManager was attempted and successful than it will publish an AuthenticationSuccessEvent
    // This check prevents a duplicate AuthenticationSuccessEvent if the parent AuthenticationManager already published it
    if (parentResult == null) {
        eventPublisher.publishAuthenticationSuccess(result);
    }
    return result;
}
```

其中，有一个防止重复发布 AuthenticationSuccessEvent 事件的处理，即 parentResult 为空。如果 parentResult 为 null，则代表父 AuthenticationManager 不存在或者没有身份认证成功，也即没有发布过 AuthenticationSuccessEvent 事件。此时，便由此处发布 AuthenticationSuccessEvent 事件。

最后，便是对于 `lastException` 的相关处理。

如果 `lastException` 为 `null`，则代表当前的 `Authentication` 并没有对应支持的 Provider。此时，便会抛出相应异常。

```java
if (lastException == null) {
    lastException = new ProviderNotFoundException(messages.getMessage(
        "ProviderManager.providerNotFound",
        new Object[] { toTest.getName() },
        "No AuthenticationProvider found for {0}"));
}
```

接下来，如同防止重复发布 AuthenticationSuccessEvent 事件的处理一样，也有一个防止 AbstractAuthenticationFailureEvent 事件重复发布的逻辑处理。如果 parentException 为 null，则代表父AuthenticationManager 不存在、没有进行身份认证或者发布过 AbstractAuthenticationFailureEvent 事件，此时，便由此处发布 AbstractAuthenticationFailureEvent 事件。

```java
if (parentException == null) {
    prepareException(lastException, authentication);
}
​
throw lastException;
```

最后，抛出 `lastException`。

但是，抛出 `lastException` 之后呢？其实，是被另外一个 Filter 捕获并初始化到当前用户的 `Request` 中，后续将讲解。

## 17. 添加验证码

1. 添加依赖

使用验证码需要使用第三方依赖

这里使用hutool 工具包。

hutool 中有我们本次需要使用的一个验证码算法类，集成后，可以直接使用，比较方便。

```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.4.2</version>
</dependency>
```

2. 生成验证码

添加完hutool依赖以后，接下来，就需要使用其中的验证码算法类生成验证码，并将具体的验证码存储起来，方便Spring Security 框架相关Filter后续验证。

```java
@GetMapping("/captcha/generate")
public void captchaGenerate(HttpSession session, HttpServletResponse response) {
    response.setHeader("Pragma", "No-cache");
    response.setHeader("Cache-Control", "no-cache");
    response.setDateHeader("Expires", 0);
    response.setContentType("image/jpeg");
​
    CircleCaptcha captcha = CaptchaUtil.createCircleCaptcha(80, 40, 4, 25);
    try {
        ServletOutputStream outputStream = response.getOutputStream();
​
        // 图形验证码写出既输出到流，当然也可以输出到文件，如captcha.write("d:/circle_captcha.jpeg");
        captcha.write(outputStream);
​
        // 从带有圆圈类型的图形验证码图片中获取其中的字符串验证码
        // 注意，获取字符串验证码要在图形验证码write后，不然得到的值为null
        String captchaCode = captcha.getCode();
​
        // 将字符串验证码保存到session中
        session.setAttribute("captcha", captchaCode);
​
        logger.info("session id {}， 生成的验证码 {}", session.getId(), captchaCode);
​
        //关闭流
        outputStream.close();
    } catch (IOException e) {
        logger.error(e.getMessage(), e);
    }
}
```

这里直接使用 hutool 的 CircleCaptcha 类生成验证码图片，生成完成后，把验证码存入Session，供后续Spring Security Filter 获取并验证。

注意，获取字符串验证码要在图形验证码write后，不然得到的值为null。

3. **登录页面改造**

根据前面已经完成的验证码生成逻辑，相应的改造一下登录页面，添加验证码表单项。

```html
<div class="login-form">
  <form th:action="@{/login}" method="post" th:method="post" class="mt-1">
    <div class="form-group">
      <input type="text" class="form-control" name="username" placeholder="用户名">
    </div>
    <div class="form-group">
      <input type="password" class="form-control" name="password" placeholder="密码">
    </div>
    <div class="form-row">
      <div class="form-group col-md-10">
        <input type="text" class="form-control" name="captcha" placeholder="验证码">
      </div>
      <div class="form-group col-md-2">
        <a href="####" id="captcha_link">
          <img id="captcha_img" src="../static/img/captcha.jpg" th:src="@{/captcha/generate}" /></a>
      </div>
    </div>
    <div class="checkbox">
      <label><input type="checkbox"> 记住我</label>
    </div>
    <button type="submit" class="btn btn-primary btn-block mb-1 mt-1">登录</button>
    <p class="text-muted text-center"> <a href="login.html#">
      <small>忘记密码了？</small></a> | <a href="#">注册一个新账号</a>
    </p>
  </form>
</div>
```

**注意，此处的验证码是可以点击的，在点击之后，会刷新验证码，也就是重新生成一次，防止看不清楚的情况发生。**

4. **自定义Filter**

Spring Security 框架默认的 UsernamePasswordAuthenticationFilter 中并没有针对验证码的处理，只有用户名和密码。因此，我们需要自定义一个包含验证码验证的Filter。

```java
public class UsernamePasswordCaptchaAuthenticationFilter extends AbstractCaptchaAuthenticationProcessingFilter {
​
    ......
​
    @Override
    public void captchaAuthentication(HttpServletRequest request) throws AuthenticationException, IOException, ServletException {
        if (postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException(
                    "Authentication method not supported: " + request.getMethod());
        }
​
        String captcha = obtainCaptcha(request);
        String captchaFromRequest = (String) request.getSession(false).getAttribute("captcha");
​
        if (captcha == null) {
            throw new EmptyCaptchaAuthenticationException("the captcha is null.");
        }
​
        if (captchaFromRequest == null) {
            throw new InternalAuthenticationServiceException("the captcha generator occurred error.");
        }
​
        boolean captchaMatched = captchaCaseSensitive ? Objects.equals(captcha, captchaFromRequest)
                : Objects.equals(captcha.toLowerCase(), captchaFromRequest.toLowerCase());
​
        if (!captchaMatched) {
            throw new BadCaptchaAuthenticationException("the captcha is not matched.");
        }
    }
​
  ......
​
}
```

在此 Filter 中，绝大部分逻辑，都与 Spring Security 框架默认的 UsernamePasswordAuthenticationFilter 相同，只添加了验证码相关验证逻辑。

注意，关于验证码的验证逻辑，此处使用抛出异常的方式来实现，而不是单纯的返回 true/false。抛出异常后，在基类的 doFilter 方法中，即可捕获，并做进一步处理。

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
​
    ......
​
        try {
            captchaAuthentication(request);
        }
    catch (InternalAuthenticationServiceException failed) {
        logger.error(
            "An internal error occurred while trying to authenticate the captcha.",
            failed);
        unsuccessfulAuthentication(request, response, failed);
​
        return;
    }
    catch (AuthenticationException failed) {
        // Authentication failed
        unsuccessfulAuthentication(request, response, failed);
​
        return;
    }
​
    ......
}
```

5. **Spring Security 配置改造**

既然自定义了 UsernamePasswordCaptchaAuthenticationFilter，那么势必要配置到 Spring Security 中，下面，就针对原有的 Spring Security 配置进行改造。

```java
@EnableWebSecurity
@Configuration
public class SpringSecurityConfiguration extends WebSecurityConfigurerAdapter {
​
    ......
​
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        ......
​
        http.addFilterAt(usernamePasswordAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        http.addFilterAfter(customFilterSecurityInterceptor(), FilterSecurityInterceptor.class);
    }
​
    private UsernamePasswordCaptchaAuthenticationFilter usernamePasswordAuthenticationFilter() throws Exception {
        UsernamePasswordCaptchaAuthenticationFilter authenticationFilter = new UsernamePasswordCaptchaAuthenticationFilter();
        authenticationFilter.setAuthenticationSuccessHandler(authenticationSuccessHandler());
        authenticationFilter.setAuthenticationFailureHandler(authenticationFailureHandler());
        authenticationFilter.setAuthenticationManager(authenticationManager());
        return authenticationFilter;
    }
​
    private AuthenticationProvider daoAuthenticationProvider() {
        DaoAuthenticationProvider daoAuthenticationProvider = new DaoAuthenticationProvider();
        daoAuthenticationProvider.setUserDetailsService(customJdbcUserDetailsService());
        daoAuthenticationProvider.setPasswordEncoder(new BCryptPasswordEncoder());
        return daoAuthenticationProvider;
    }
​
    private AuthenticationSuccessHandler authenticationSuccessHandler() {
        SavedRequestAwareAuthenticationSuccessHandler authenticationSuccessHandler = new SavedRequestAwareAuthenticationSuccessHandler();
        authenticationSuccessHandler.setDefaultTargetUrl("/index");
        return authenticationSuccessHandler;
    }
​
    private AuthenticationFailureHandler authenticationFailureHandler() {
        SimpleUrlAuthenticationFailureHandler authenticationFailureHandler = new SimpleUrlAuthenticationFailureHandler();
        authenticationFailureHandler.setDefaultFailureUrl("/login_fail");
        return authenticationFailureHandler;
    }
​
  ......
​
}
```

可以看到，无论是前面讲过的 CA登录 方式，还是既有的 用户名密码登录 方式改造，都需要配置一整套的逻辑，如 Filter、AuthenticationProvider、AuthenticationSuccessHandler、AuthenticationFailureHandler，当然，如果是全新的登录方式（如CA登录），还需要自定义相应的 Authentication 实现，即 token 等。

## 18. ExceptionTranslationFilter

这是 **Spring Security** 框架 **FilterChain** 中倒数第二个 Filter，承载着承上启下的作用。

如当用户未登录时抛出的 AccessDeniedException 异常、或者其它 AuthenticationException 异常。此时，系统会跳转到固定的页面，如 403 页面、或者执行其它操作。

这是怎么实现的呢？

其实，这就是 ExceptionTranslationFilter 实现的。

**此 Filter 用于处理任何 AccessDeniedException 和 AuthenticationException 异常，** **承担着 Java 异常 和 HTTP 状态码之间的桥梁作用，并没有任何其它的实质性作用。**

如果发生 AuthenticationException 异常，此 Filter 会使用 AuthenticationEntryPoint 来通用处理任何 AbstractSecurityInterceptor 子类出现的认证失败逻辑。

首先，就是捕获后续 Filter 发生的异常，然后，解析异常类型：AuthenticationException、AccessDeniedException。

```java
catch (Exception ex) {
    // Try to extract a SpringSecurityException from the stacktrace
    Throwable[] causeChain = throwableAnalyzer.determineCauseChain(ex);
    RuntimeException ase = (AuthenticationException) throwableAnalyzer
        .getFirstThrowableOfType(AuthenticationException.class, causeChain);
​
    if (ase == null) {
        ase = (AccessDeniedException) throwableAnalyzer.getFirstThrowableOfType(
            AccessDeniedException.class, causeChain);
    }
​
    if (ase != null) {
        if (response.isCommitted()) {
            throw new ServletException("Unable to handle the Spring Security Exception because the response is already committed.", ex);
        }
        handleSpringSecurityException(request, response, chain, ase);
    }
    else {
        // Rethrow ServletExceptions and RuntimeExceptions as-is
        if (ex instanceof ServletException) {
            throw (ServletException) ex;
        }
        else if (ex instanceof RuntimeException) {
            throw (RuntimeException) ex;
        }
​
        // Wrap other Exceptions. This shouldn't actually happen
        // as we've already covered all the possibilities for doFilter
        throw new RuntimeException(ex);
    }
}
```

然后调用 handleSpringSecurityException 方法处理异常。

如果是 AuthenticationException 异常，则调用 AuthenticationEntryPoint 来处理。

```java
private void handleSpringSecurityException(HttpServletRequest request,
      HttpServletResponse response, FilterChain chain, RuntimeException exception)
      throws IOException, ServletException {
    if (exception instanceof AuthenticationException) {
        logger.debug(
            "Authentication exception occurred; redirecting to authentication entry point",
            exception);
​
        sendStartAuthentication(request, response, chain,
                                (AuthenticationException) exception);
    }
    
  ......
}
```

如果是 AccessDeniedException 异常，先判断是否是未登录状态或者是记住我状态，如果是这两种状态，则调用 AuthenticationEntryPoint 来通用处理。

```java
private void handleSpringSecurityException(HttpServletRequest request,
      HttpServletResponse response, FilterChain chain, RuntimeException exception)
      throws IOException, ServletException {
    if (exception instanceof AuthenticationException) {
        ......
    }
    else if (exception instanceof AccessDeniedException) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authenticationTrustResolver.isAnonymous(authentication) || authenticationTrustResolver.isRememberMe(authentication)) {
            logger.debug(
                "Access is denied (user is " + (authenticationTrustResolver.isAnonymous(authentication) ? "anonymous" : "not fully authenticated") + "); redirecting to authentication entry point",
                exception);
​
            sendStartAuthentication(
                request,
                response,
                chain,
                new InsufficientAuthenticationException(
                    messages.getMessage(
                        "ExceptionTranslationFilter.insufficientAuthentication",
                        "Full authentication is required to access this resource")));
        }
    ......  
    }
}
```

这两种情况都调用了 sendStartAuthentication 方法来处理，而其逻辑便是先清空失效的 Authentication，然后再调用 AuthenticationEntryPoint 来通用处理异常。

```java
protected void sendStartAuthentication(HttpServletRequest request,
      HttpServletResponse response, FilterChain chain,
      AuthenticationException reason) throws ServletException, IOException {
    // SEC-112: Clear the SecurityContextHolder's Authentication, as the
    // existing Authentication is no longer considered valid
    SecurityContextHolder.getContext().setAuthentication(null);
    requestCache.saveRequest(request, response);
    logger.debug("Calling Authentication entry point.");
    authenticationEntryPoint.commence(request, response, reason);
}
```

如果发生未 AccessDeniedException 异常时，既不是登录状态，也不是是记住我状态，则会调用 AccessDeniedHandler 来处理异常。

```java
private void handleSpringSecurityException(HttpServletRequest request,
      HttpServletResponse response, FilterChain chain, RuntimeException exception)
      throws IOException, ServletException {
    if (exception instanceof AuthenticationException) {
      ......
    }
    else if (exception instanceof AccessDeniedException) {
      ......
      }
      else {
        logger.debug(
            "Access is denied (user is not anonymous); delegating to AccessDeniedHandler",
            exception);
​
        accessDeniedHandler.handle(request, response,
            (AccessDeniedException) exception);
      }
    }
  }
```

AccessDeniedHandler 默认使用 AccessDeniedHandlerImpl 实现，会根据情况跳转到错误页面，或者发送对应的HTTP状态码。**这就是文章开头所说的介于Java异常和HTTP状态码之间的桥梁作用**。

```java
public void handle(HttpServletRequest request, HttpServletResponse response,
      AccessDeniedException accessDeniedException) throws IOException,
      ServletException {
    if (!response.isCommitted()) {
      if (errorPage != null) {
        // Put exception into request scope (perhaps of use to a view)
        request.setAttribute(WebAttributes.ACCESS_DENIED_403,
            accessDeniedException);
​
        // Set the 403 status code.
        response.setStatus(HttpStatus.FORBIDDEN.value());
​
        // forward to error page.
        RequestDispatcher dispatcher = request.getRequestDispatcher(errorPage);
        dispatcher.forward(request, response);
      }
      else {
        response.sendError(HttpStatus.FORBIDDEN.value(),
          HttpStatus.FORBIDDEN.getReasonPhrase());
      }
    }
  }
```

## 19. SecurityContextPersistenceFilter

**SecurityContextPersistenceFilter**，为 **Spring Security** 框架 **Filter Chain** 过滤器链的第一个 Filter，具有不可或缺的作用。

记得 Spring Security 框架中，可以通过 HttpServletRequest 中的 SPRING_SECURITY_CONTEXT 属性，直接获取当前登录用户的 SecurityContext 吗？其正是由此 Filter 实现的，而此也正是该类的重要使命。

```java
@RequestMapping("/user")
@ResponseBody
public Object user(HttpServletRequest request) {
    return request.getSession().getAttribute(HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY);
}
```

启动系统，正常登录后，访问 http://localhost:8080/springsecuritylearning/user 路径，则能正常展示当前已身份认证成功的 SecurityContext。

![img](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20211009152037.png)

下面，将会详细分析此 Filter 的具体功用。

该 Filter 从配置的 **SecurityContextRepository**（默认为 **HttpSessionSecurityContextRepository**）中获取信息并在请求完成后将其存储到 **SecurityContextHolder** 中。

并且，该 Filter 必须在其它任何身份验证处理之前执行，这也就是为什么会是 **Spring Security** 框架 **Filter Chain** 过滤器链的第一个 Filter。

同 FilterSecurityInterceptor 一样，该 Filter 仅会执行一次。

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
      throws IOException, ServletException {
    ......
​
    if (request.getAttribute(FILTER_APPLIED) != null) {
        // ensure that filter is only applied once per request
        chain.doFilter(request, response);
        return;
    }
​
  ......
}
```

在其它 Filter 执行之前，会先从 SecurityContextRepository 中获取当前的 SecurityContext，然后再执行后续 Filter。

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
      throws IOException, ServletException {
    ......
    HttpRequestResponseHolder holder = new HttpRequestResponseHolder(request,
        response);
    SecurityContext contextBeforeChainExecution = repo.loadContext(holder);
​
    try {
      SecurityContextHolder.setContext(contextBeforeChainExecution);
​
      chain.doFilter(holder.getRequest(), holder.getResponse());
​
    }
    ......
  }
```

从 SecurityContextRepository 中获取当前的 SecurityContext 的逻辑如下。

```java
public SecurityContext loadContext(HttpRequestResponseHolder requestResponseHolder) {
    HttpServletRequest request = requestResponseHolder.getRequest();
    HttpServletResponse response = requestResponseHolder.getResponse();
    HttpSession httpSession = request.getSession(false);
​
    SecurityContext context = readSecurityContextFromSession(httpSession);
​
    if (context == null) {
        if (logger.isDebugEnabled()) {
            logger.debug("No SecurityContext was available from the HttpSession: "
                         + httpSession + ". " + "A new one will be created.");
        }
        context = generateNewContext();
​
    }
    
    ......
​
    return context;
}
```

如果 SecurityContext 不存在，则会创建一个新的 SecurityContext 并返回。

待后续 Filter 执行完毕，SecurityContextRepository 会将当前新的 SecurityContext 进行保存，并且清空当前的SecurityContextHolder 中的 SecurityContext。

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
      throws IOException, ServletException {
    ......
​
    try {
       ......
​
    }
    finally {
        SecurityContext contextAfterChainExecution = SecurityContextHolder
            .getContext();
        // Crucial removal of SecurityContextHolder contents - do this before anything
        // else.
        SecurityContextHolder.clearContext();
        repo.saveContext(contextAfterChainExecution, holder.getRequest(),
                         holder.getResponse());
        request.removeAttribute(FILTER_APPLIED);
​
        ......
    }
}
```

最后，SecurityContextRepository 会保存新的 SecurityContext，并清空 FILTER_APPLIED 标识。SecurityContextRepository 保存新的 SecurityContext 逻辑如下。

```java
public void saveContext(SecurityContext context, HttpServletRequest request,
      HttpServletResponse response) {
    SaveContextOnUpdateOrErrorResponseWrapper responseWrapper = WebUtils
        .getNativeResponse(response,
                           SaveContextOnUpdateOrErrorResponseWrapper.class);
    if (responseWrapper == null) {
        throw new IllegalStateException(
            "Cannot invoke saveContext on response "
            + response
            + ". You must use the HttpRequestResponseHolder.response after invoking loadContext");
    }
    // saveContext() might already be called by the response wrapper
    // if something in the chain called sendError() or sendRedirect(). This ensures we
    // only call it
    // once per request.
    if (!responseWrapper.isContextSaved()) {
        responseWrapper.saveContext(context);
    }
}
```

方法开头所获取的 SaveContextOnUpdateOrErrorResponseWrapper（SaveToSessionResponseWrapper 父类），正是在 Filter 执行开始时，loadContext 方法所设置的 response。

```java
public SecurityContext loadContext(HttpRequestResponseHolder requestResponseHolder) {
    HttpServletRequest request = requestResponseHolder.getRequest();
    HttpServletResponse response = requestResponseHolder.getResponse();
    HttpSession httpSession = request.getSession(false);
​
    ......
​
    SaveToSessionResponseWrapper wrappedResponse = new SaveToSessionResponseWrapper(
        response, request, httpSession != null, context);
    requestResponseHolder.setResponse(wrappedResponse);
​
    ......
​
    return context;
}
```

最后，新的 SecurityContext 的保存逻辑，会落到 SaveContextOnUpdateOrErrorResponseWrapper 的子类上，即内部类 HttpSessionSecurityContextRepository.SaveToSessionResponseWrapper。

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20211009180425.png" alt="image-20211009180425196" style="zoom:50%;" />

保存 SecurityContext 逻辑如下。

```java
protected void saveContext(SecurityContext context) {
    final Authentication authentication = context.getAuthentication();
    HttpSession httpSession = request.getSession(false);
​
    // See SEC-776
    if (authentication == null || trustResolver.isAnonymous(authentication)) {
        if (logger.isDebugEnabled()) {
            logger.debug("SecurityContext is empty or contents are anonymous - context will not be stored in HttpSession.");
        }
​
        if (httpSession != null && authBeforeExecution != null) {
            // SEC-1587 A non-anonymous context may still be in the session
            // SEC-1735 remove if the contextBeforeExecution was not anonymous
            httpSession.removeAttribute(springSecurityContextKey);
        }
        return;
    }
​
    if (httpSession == null) {
        httpSession = createNewSessionIfAllowed(context);
    }
​
    // If HttpSession exists, store current SecurityContext but only if it has
    // actually changed in this thread (see SEC-37, SEC-1307, SEC-1528)
    if (httpSession != null) {
        // We may have a new session, so check also whether the context attribute
        // is set SEC-1561
        if (contextChanged(context)
            || httpSession.getAttribute(springSecurityContextKey) == null) {
            httpSession.setAttribute(springSecurityContextKey, context);
​
            if (logger.isDebugEnabled()) {
                logger.debug("SecurityContext '" + context
                             + "' stored to HttpSession: '" + httpSession);
            }
        }
    }
}
```

其实，就是在判断当前的 SecurityContext 有无变化，或者其中的 Authentication 有无变化，来清除或者设置HttpSession 中的 SPRING_SECURITY_CONTEXT 属性。比如，身份认证成功、身份认证失败等。

**身份认证成功后，系统会设置 SPRING_SECURITY_CONTEXT 属性值到 HttpSession 中。这也是为什么我们可以通过 Request 来获取** **SPRING_SECURITY_CONTEXT 标识值，也即** **SecurityContext** **的原因**。



## 20. SecurityContextHolder及SecurityContextHolderStrategy

如何静态的、较为方便的获取当前系统已登录的用户的信息？这恐怕就要靠 Spring Security 框架的另外一个“著名”的组件类 **SecurityContextHolder** 了。

```java
/**
 * Associates a given {@link SecurityContext} with the current execution thread.
 */
```

**SecurityContextHolder** 类最核心的作用，便是将当前给定的 SecurityContext 与 当前执行线程绑定，从而方便后续直接获取。

其提供了一系列静态方法，从而可以保证能在各处随心所欲的调用。

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20211009182243.png" alt="image-20211009182243224" style="zoom:50%;" />

该类中还有一个比较重要的属性：SecurityContextHolderStrategy。

上面已经说了，**SecurityContextHolder** 类最核心的作用，便是将当前给定的 SecurityContext 与 当前执行线程绑定。那么，如何绑定，便由 SecurityContextHolderStrategy 类负责。

SecurityContextHolderStrategy 接口共有三种实现。

其一，也是默认的实现，ThreadLocalSecurityContextHolderStrategy。顾名思义，基于 **ThreadLocal** 的接口实现。

```java
final class ThreadLocalSecurityContextHolderStrategy implements
    SecurityContextHolderStrategy {
  // ~ Static fields/initializers
  // =====================================================================================
​
  private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<>();
```

其二，InheritableThreadLocalSecurityContextHolderStrategy，有继承关系的 **ThreadLocal 的子类** **InheritableThreadLocal** 的实现*。*

```java
final class InheritableThreadLocalSecurityContextHolderStrategy implements
    SecurityContextHolderStrategy {
  // ~ Static fields/initializers
  // =====================================================================================
​
  private static final ThreadLocal<SecurityContext> contextHolder = new InheritableThreadLocal<>();
```

**InheritableThreadLocal** 相较于 **ThreadLocal**，多了子线程可以继承父线程的属性的特性，但是，针对普通WEB应用，应该是英雄无用武之地。

其三，GlobalSecurityContextHolderStrategy，只有一个静态 SecurityContext 属性的接口实现。

```java 
final class GlobalSecurityContextHolderStrategy implements SecurityContextHolderStrategy {
  // ~ Static fields/initializers
  // =====================================================================================
​
  private static SecurityContext contextHolder;
```

由于其只有一个静态的 SecurityContext 属性，这就决定了其使用范围比较小众，如C/S客户端应用，如Swing。

```java
/**
 * A <code>static</code> field-based implementation of
 * {@link SecurityContextHolderStrategy}.
 * <p>
 * This means that all instances in the JVM share the same <code>SecurityContext</code>.
 * This is generally useful with rich clients, such as Swing.
 */
```

**SecurityContextHolder** 类有2种方式初始化 **SecurityContextHolderStrategy**。

其一，通过静态方法 **setStrategyName**。

```java
public static void setStrategyName(String strategyName) {
    SecurityContextHolder.strategyName = strategyName;
    initialize();
}
```

其二，通过设置属性值 **spring.security.strategy**。

```java
public static final String SYSTEM_PROPERTY = "spring.security.strategy";
private static String strategyName = System.getProperty(SYSTEM_PROPERTY);
```

具体初始化逻辑如下，默认为 ThreadLocalSecurityContextHolderStrategy。

```java
private static void initialize() {
    if (!StringUtils.hasText(strategyName)) {
        // Set default
        strategyName = MODE_THREADLOCAL;
    }
​
    if (strategyName.equals(MODE_THREADLOCAL)) {
        strategy = new ThreadLocalSecurityContextHolderStrategy();
    }
    else if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {
        strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
    }
    else if (strategyName.equals(MODE_GLOBAL)) {
        strategy = new GlobalSecurityContextHolderStrategy();
    }
    else {
        // Try to load a custom strategy
        try {
            Class<?> clazz = Class.forName(strategyName);
            Constructor<?> customStrategy = clazz.getConstructor();
            strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();
        }
        catch (Exception ex) {
            ReflectionUtils.handleReflectionException(ex);
        }
    }
​
    initializeCount++;
}
```

如果没有设置系统属性 **spring.security.strategy**，也没有通过静态方法 **setStrategyName** 设置 strategyName，那么系统则会默认创建 **ThreadLocalSecurityContextHolderStrategy** 实例对象。

初始化完成后，可通过 setContext 方法将 SecurityContext 与 当前线程绑定。

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20211009182904.png" alt="image-20211009182904116" style="zoom:50%;" />

也可通过 getContext 方法获取与当前线程绑定的 SecurityContext。

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20211009182959.png" alt="image-20211009182959587" style="zoom:50%;" />

如在 Controller 中，获取登录后的 SecurityContext。

```java
@RequestMapping("/user")
@ResponseBody
public Object user() {
    return SecurityContextHolder.getContext();
}
```

启动系统，正常登录后，访问 http://localhost:8080/springsecuritylearning/user 路径，则能正常展示当前已身份认证成功的 SecurityContext。

![img](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20211009183101.png)

## 21. SpringSecurity工作流程图解

![image-20211009225622791](https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20211009225622.png)





## 3. SpringSecurity Web权限验证

### 3.1 使用数据库验证密码

> UserDetailsService接口，查询数据库校验密码
>
> PasswordEncoder接口，密码加密

1. 创建用户表



1. 创建配置类，设置使用哪个userDetailsService实现类

~~~java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(password());
    }

    @Bean
    PasswordEncoder password(){
        return new BCryptPasswordEncoder(); // 指定密码加密方式
    }
}
~~~

2. 编写实现类，返回user对象，user对象有用户名密码和操作权限 

~~~java
@Service("userDetailsService ")
public class MyUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
        return new User("root",new BCryptPasswordEncoder().encode("root"),auths); //应该从数据库拿，这里演示用
    }
}
~~~

### 3.2 hasAuthority()方法-单一权限

如果当前主体有指定的权限返回ture，否则返回false

**步骤**

1. 在配置类设置当前访问地址有哪些权限

~~~ Java
.antMatchers("/test/index").hasAuthority("admins")  // 具有admin权限才能访问这个路径
~~~

2. 在UserDetailsService，把返回User对象设置权限

~~~java
List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("admins");
~~~

 

### 3.3 hasAnyAuthority()方法-多个权限

配置步骤同上

### 3.4 hasRole()方法

如果用户具备给定角色，返回ture，否则返回403

示例：

~~~java
.antMatchers("/test/index").hasRole("sale")
~~~

### 3.5 hasAnyRole()方法

 有其中任一角色即可访问



## 4. SpringSecurity源码

 ### 4.1 本质上是一个过滤器链



~~~java
FilterSecurityInterceptor //方法级权限过滤器，位于过滤链的最底部
~~~



~~~java
ExceptionTranslationFilter   //异常过滤器，用来处理在认证授权过程中抛出的异常
~~~



~~~java
UsernamePasswordAuthenticationFilter //对/login的post请求做拦截，校验表单中的用户名密码
~~~

### 4.2 过滤器如何加载的

#### 4.2.1 配置SpringSecurity的过滤器

~~~java
DelegatingFilterProxy //类，Proxy for a standard Servlet Filter，代理Servlet Filter 
~~~

 其中的方法

~~~java
	@Override
	protected void initFilterBean() throws ServletException {
		synchronized (this.delegateMonitor) {
			if (this.delegate == null) {
				// If no target bean name specified, use filter name.
				if (this.targetBeanName == null) {
					this.targetBeanName = getFilterName();
				}
				// Fetch Spring root application context and initialize the delegate early,
				// if possible. If the root application context will be started after this
				// filter proxy, we'll have to resort to lazy initialization.
				WebApplicationContext wac = findWebApplicationContext();
				if (wac != null) {
					this.delegate = initDelegate(wac);   //关键，初始化方法
				}
			}
		}
	}
~~~

调用了方法initDelegate，去初始化过滤器链，其中拿到的beanName是FilterChainProxy

~~~java
	protected Filter initDelegate(WebApplicationContext wac) throws ServletException {
		String targetBeanName = getTargetBeanName();   //会拿到FilterChainProxy
		Assert.state(targetBeanName != null, "No target bean name set");
		Filter delegate = wac.getBean(targetBeanName, Filter.class);
		if (isTargetFilterLifecycle()) {
			delegate.init(getFilterConfig());
		}
		return delegate;
	}
~~~

FilterChainProxy类的方法

~~~java
	@Override
	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
		boolean clearContext = request.getAttribute(FILTER_APPLIED) == null;
		if (clearContext) {
			try {
				request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
				doFilterInternal(request, response, chain);
			}
			finally {
				SecurityContextHolder.clearContext();
				request.removeAttribute(FILTER_APPLIED);
			}
		}
		else {
			doFilterInternal(request, response, chain);  //重点
		}
	}
~~~

~~~java
	private void doFilterInternal(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {

		FirewalledRequest fwRequest = firewall
				.getFirewalledRequest((HttpServletRequest) request);
		HttpServletResponse fwResponse = firewall
				.getFirewalledResponse((HttpServletResponse) response);

		List<Filter> filters = getFilters(fwRequest); //获取所有过滤器

		if (filters == null || filters.size() == 0) {
			if (logger.isDebugEnabled()) {
				logger.debug(UrlUtils.buildRequestUrl(fwRequest)
						+ (filters == null ? " has no matching filters"
								: " has an empty filter list"));
			}

			fwRequest.reset();

			chain.doFilter(fwRequest, fwResponse);

			return;
		}

		VirtualFilterChain vfc = new VirtualFilterChain(fwRequest, chain, filters);
		vfc.doFilter(fwRequest, fwResponse);
	}
~~~

### 4.3 认证流程

常用的认证方式是UsernamePasswordAuthenticationFilter

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210919115045.png" alt="image-20210919115045695" style="zoom:50%;" />

#### 4.3.1 AbstractAuthenticationProcessingFilter过滤器

该过滤器的父类是AbstractAuthenticationProcessingFilter

父类中对应有doFilter()方法

执行步骤：

1. 判断是否是post方法，是就过滤，不是就放行

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210919115707.png" alt="image-20210919115707797" style="zoom:50%;" />

2. 调用子类的方法进行认证，认证成功后，把认证信息封装到对象中

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210919120038.png" alt="image-20210919120037983" style="zoom:50%;" />

3. session策略处理

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210920103347.png" alt="image-20210920103347071" style="zoom:50%;" />

4. 认证失败抛出异常，执行认证失败的方法

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210920103636.png" alt="image-20210920103636765" style="zoom:50%;" />

5. 认证成功，执行认证成功的方法

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210920103921.png" alt="image-20210920103921503" style="zoom:50%;" />

**第二步中调用子类方法认证的实现**

~~~java
authResult = attemptAuthentication(request, response);
~~~

其子类为：

```java
UsernamePasswordAuthenticationFilter
```

子类的方法实现为

~~~java
	public Authentication attemptAuthentication(HttpServletRequest request,
			HttpServletResponse response) throws AuthenticationException {
		if (postOnly && !request.getMethod().equals("POST")) {   // 1. 判断是否是post提交，不是就抛出异常
			throw new AuthenticationServiceException(
					"Authentication method not supported: " + request.getMethod());
		}

		String username = obtainUsername(request); // 2. 获取表单提交的数据
		String password = obtainPassword(request);

		if (username == null) {
			username = "";
		}

		if (password == null) {
			password = "";
		}

		username = username.trim();

		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
				username, password); // 3. 使用表单数据，构造token对象，编辑为未认证状态，

		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);   // 把请求属性设置到对象里

		return this.getAuthenticationManager().authenticate(authRequest); // 调用方法进行身份认证（UserDetailsService）
	}

~~~

#### 4.3.2 UsernamePasswordAuthenticationToken的构造过程

UsernamePasswordAuthenticationToken.class类的两个构造方法

未认证的方法

~~~java
	public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
		super(null);
		this.principal = principal;
		this.credentials = credentials;
		setAuthenticated(false);
	}
~~~

 已经认证的方法

~~~Java
	public UsernamePasswordAuthenticationToken(Object principal, Object credentials,
			Collection<? extends GrantedAuthority> authorities) {
		super(authorities);
		this.principal = principal;
		this.credentials = credentials;
		super.setAuthenticated(true); // must use super, as we override
	}
~~~

该类实现了接口Authentication

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210921113604.png" alt="image-20210921113604867" style="zoom:50%;" />

#### 4.3.3 ProviderManager

提供认证方法

~~~java
	public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		boolean debug = logger.isDebugEnabled();

		for (AuthenticationProvider provider : getProviders()) {
			if (!provider.supports(toTest)) {
				continue;
			}

			try {
				result = provider.authenticate(authentication);

				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
		}


		if (result != null) {
			if (eraseCredentialsAfterAuthentication
					&& (result instanceof CredentialsContainer)) {
				// Authentication is complete. Remove credentials and other secret data
				// from authentication
				((CredentialsContainer) result).eraseCredentials();
			}

			// If the parent AuthenticationManager was attempted and successful than it will publish an AuthenticationSuccessEvent
			// This check prevents a duplicate AuthenticationSuccessEvent if the parent AuthenticationManager already published it
			if (parentResult == null) {
				eventPublisher.publishAuthenticationSuccess(result);
			}
			return result;
		}

		// Parent was null, or didn't authenticate (or throw an exception).

		if (lastException == null) {
			lastException = new ProviderNotFoundException(messages.getMessage(
					"ProviderManager.providerNotFound",
					new Object[] { toTest.getName() },
					"No AuthenticationProvider found for {0}"));
		}

		// If the parent AuthenticationManager was attempted and failed than it will publish an AbstractAuthenticationFailureEvent
		// This check prevents a duplicate AbstractAuthenticationFailureEvent if the parent AuthenticationManager already published it
		if (parentException == null) {
			prepareException(lastException, authentication);
		}

		throw lastException;
	}
~~~





## 5. 自定义403页面

步骤：

1. 编写页面，放在static文件夹下

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210919105122.png" alt="image-20210919105122120" style="zoom:50%;" />

2. 在配置类中进行配置

~~~java
// 配置没有权限访问跳转自定义页面
http.exceptionHandling().accessDeniedPage("unauth.html");
~~~

效果

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210919105509.png" alt="image-20210919105509571" style="zoom:50%;" />

## 6. 注解

### 6.1 @Secured

判断是否具有角色，匹配的字符串需要添加前缀“ROLE_”

使用注解先要开启注解功能

~~~Java
@EnableGlobalMethodSecurity(securedEnabled = true)
~~~

### 6.2 @PreAuthorize("hasAuthority('...')")

在方法进入前进行权限验证

需要先开启注解功能

~~~java
@EnableGlobalMethodSecurity(prePostEnabled = true)
~~~

使用

<img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210919110330.png" alt="image-20210919110329995" style="zoom:50%;" />

### 6.3 @PostAuthorize("hasAuthority('...')")

在方法执行之后进行权限验证，实际使用不多

## 7. 用户注销

1. 在配置类中调价退出的配置

~~~java
 // 退出
http.logout().logoutUrl("/logout").logoutSuccessUrl("/test/hello").permitAll();
~~~

2. 在页面添加退出超链接

   <img src="https://cdn.jsdelivr.net/gh/wangfei0/picgo-repository@main//img/20210919113353.png" alt="image-20210919113353870" style="zoom:50%;" />

注意：不需要写/logout的Controller，SpringSecurity会配置好

## 8. 记住我

基于数据库的记住我

基本不会用到，用到了再看吧

