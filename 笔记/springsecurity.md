# SpringSecurity

特点 | Spring Security | Apache Shiro
--- | --- | ---
适用范围	| 适用于大型、复杂的企业级应用程序	| 适用于各种规模的Java应用程序，包括Web和非Web应用程序
框架集成性	| 与Spring框架深度集成，可以利用Spring的各种功能和模块	| 独立框架，可以与各种Java应用程序集成
身份验证和授权	| 提供全面的身份验证和授权功能，支持多种认证方式和授权策略	| 提供基本的身份验证和授权功能，较为灵活的自定义权限管理
安全配置	| 使用基于过滤器链的方式进行安全配置，通过配置类进行细粒度的配置	| 使用Ini配置文件进行安全配置，支持编程式和注解式的配置
扩展性和灵活性	| 高度可定制和灵活，支持自定义的认证和授权逻辑	| 可扩展性较强，提供许多可插拔的组件和扩展点
社区支持和生态系统	| 拥有庞大的开发社区和广泛的应用，与Spring生态系统紧密结合	| 拥有稳定的开发社区和一些扩展模块，但相对较小
学习曲线和文档资源	| 学习曲线较陡峭，但提供了丰富的官方文档、教程和社区支持	| 学习曲线相对较平缓，文档资源相对较少，但社区支持良好

## 入门案例准备工作

1. 引入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

2.原有访问受限

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307090900796.png)



## 认证

* 登录
  * 自定义登录接口，
    * 掉用ProvideManager的方法进行认证 如果认证通过该生成jwt
    * 将用户信息存入redis
  * 自定义userService在这个实现类中去查询数据库

* 校验
  * 定义jwt认证过滤器
    * 获取token
    * 解析token中的userid
    * 从redis获取用户信息
    * 存入securityContextHolder当中

## 自己配置用户

* application.xml文件配置(只能定义一个用户)

```yml
spring:
  security:
    user:
      password: ledger
      name: ledger
```



## 基于内存的多用户管理(自己配置用户在内存中)

* 创建配置类

  ```java
  @Configuration
  public class MySecurityUserConfig {
      /**
       * 存储用户信息的类
       * @return
       */
      @Bean
      public UserDetailsService userDetailsService(){
          UserDetails user1 = User.builder().username("ledger").password("123456").roles("admin").build();
          UserDetails user2 = User.builder().username("yb").password("123456").roles("admin").build();
          // InMemoryUserDetails 这个类是UserDetailsService的实现类
          InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
          //创建两个用户
          manager.createUser(user1);
          manager.createUser(user2);
          return manager;
      }
      /**
       * 加密器
       * @return
       */
      @Bean
      public PasswordEncoder passwordEncoder(){
          //没有加密的加密器
          return NoOpPasswordEncoder.getInstance();
      }
  }
  ```

  

## 密码加密处理

```java
@SpringBootTest
@Slf4j
class SpringSecutityApplicationTests {
    @Test
    void contextLoads() {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        String encode = encoder.encode("123456");
        String encode2= encoder.encode("123456");
        String encode3= encoder.encode("123456");
        boolean matches = encoder.matches("123456", encode);
        boolean matches1 = encoder.matches("123456", encode2);
        boolean matches2 = encoder.matches("123456", encode3);
        //三次加密的结果都不一样
        log.error(encode);
        log.error(encode2);
        log.error(encode3);
        assert matches&&matches1&&matches2;
    }
}
```



## 使用BCryptPasswordEncoder加密器 

```java
/**
 * 实现UserDetailsService，这个就是存储用户信息的一个类
 *
 * @author ledger
 * @version 1.0
 **/
@Configuration
public class MySecurityUserConfig {
    /**
     * 存储用户信息的类
     * @return
     */
    @Bean
    public UserDetailsService userDetailsService(){
        UserDetails user1 = User.builder().username("ledger").password(passwordEncoder().encode("123456")).roles("admin").build();
        UserDetails user2 = User.builder().username("yb").password(passwordEncoder().encode("123456")).roles("admin").build();
        // InMemoryUserDetails 这个类是UserDetailsService的实现类
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        //创建两个用户
        manager.createUser(user1);
        manager.createUser(user2);
        return manager;
    }
    /**
     * 加密器
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        //BCryptPasswordEncoder加密器
        return new BCryptPasswordEncoder();
    }
}
```

 ## 获取登录用户信息

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307091100716.png)



![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307091101549.png)



## 配置用户权限

* 这两个会出现冲突
* 后面的才会生效

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307091113667.png)

## 针对url进行访问

* 通过配置类

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * 针对url进行拦截
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
//        http.authorizeRequests()// 授权http请求
//                .anyRequest() //所有的请求
//                .denyAll(); //都拒绝访问
//        http.formLogin().permitAll(); //允许表单登录
        http.authorizeRequests()
                .mvcMatchers("/student/**") //匹配这个url
                .hasAnyAuthority("ROLE_student", "ROLE_teacher") //需要ROLE_student,ROLE_teacher这个角色
                .mvcMatchers("/teacher/**") //匹配这个url
                .hasAuthority("ROLE_teacher") //需要ROLE_teacher这个角色
                .anyRequest() //任何请求
                .authenticated(); //没有配置
        http.formLogin().permitAll(); //允许表单登录
    }
}
```

## 针对方法进行授权

* 新增注解@EnableGlobalMethodSecurity(prePostEnabled = true)  开启全局方法安全
* prePostEnabled 是指进入前拦截

```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * 针对url进行拦截
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
//        http.authorizeRequests()// 授权http请求
//                .anyRequest() //所有的请求
//                .denyAll(); //都拒绝访问
//        http.formLogin().permitAll(); //允许表单登录
        http.authorizeRequests()
                .mvcMatchers("/student/**") //匹配这个url
                .hasAnyAuthority("ROLE_student", "ROLE_teacher") //需要ROLE_student,ROLE_teacher这个角色
//                .mvcMatchers("/teacher/**") //匹配这个url
//                .hasAuthority("ROLE_teacher") //需要ROLE_teacher这个角色
                .anyRequest() //任何请求
                .authenticated(); //没有配置
        http.formLogin().permitAll(); //允许表单登录
    }
}
```

* 给方法加上权限限制

```java
@RestController
@RequestMapping("/teacher")
public class TeacherController {
    @Resource
    private TeacherService teacherService;

    @GetMapping("/query")
    @PreAuthorize("hasAuthority('teacher:query')")
    public String query(){
        return teacherService.query();
    }

    @GetMapping("/update")
    @PreAuthorize("hasAuthority('teacher:update')")
    public String update(){
        return teacherService.update();
    }

    @GetMapping("/delete")
    @PreAuthorize("hasAuthority('teacher:delete')")
    public String delete(){
        return teacherService.delete();
    }

    @GetMapping("/add")
    @PreAuthorize("hasAuthority('teacher:add')")
    public String add(){
        return teacherService.add();
    }
}

```

## 返回json(认证处理器)

* 在不同的情景下面，返回json

```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Resource
    private AppAuthenticationSuccessHandler appAuthenticationSuccessHandler;
    @Resource
    private AppAccessDeniedHandler appAccessDeniedHandler;
    @Resource
    private AppAuthenticationFailureHandler appAuthenticationFailureHandler;
    @Resource
    private AppLogoutSuccessHandler appLogoutSuccessHandler;

    /**
     * 针对url进行拦截
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest() //任何请求
                .authenticated(); //没有配置
        http.formLogin()
            	//成功登录的处理器
                .successHandler(appAuthenticationSuccessHandler)
            	//登录失败的处理器
                .failureHandler(appAuthenticationFailureHandler)
                .permitAll(); //允许表单登录
        //退出成功的处理器
        http.logout().logoutSuccessHandler(appLogoutSuccessHandler);
        //访问权限不够的处理器
        http.exceptionHandling().accessDeniedHandler(appAccessDeniedHandler);
    }
}
```

* 权限不足处理器

```java
@Component
public class AppAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException)
            throws IOException, ServletException {
        HttpResult success = HttpResult.builder().code(403).msg("fail").data("权限不足").build();
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        writer.write(JSON.toJSONString(success));
        writer.flush();
        writer.close();
    }
}

```

* 登录失败处理器

```java
@Component
@Slf4j
public class AppAuthenticationFailureHandler implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception)
            throws IOException, ServletException {
        HttpResult success = HttpResult.builder().code(400).msg("fail").data("密码不对").build();
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        writer.write(JSON.toJSONString(success));
        writer.flush();
        writer.close();
    }
}
```

* 登录成功处理器

```java
@Component
@Slf4j
public class AppAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication)
            throws IOException, ServletException {
        HttpResult success = HttpResult.builder().code(200).msg("success").data("登录成功").build();
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        writer.write(JSON.toJSONString(success));
        writer.flush();
        writer.close();

    }
}

```

* 退出登录成功处理器

```java
@Component
public class AppLogoutSuccessHandler implements LogoutSuccessHandler {
    @Override
    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication)
            throws IOException, ServletException {
        HttpResult success = HttpResult.builder().code(200).msg("fail").data("退出成功").build();
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        writer.write(JSON.toJSONString(success));
        writer.flush();
        writer.close();
    }
}
```

## 自定义用户信息

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307091437386.png)

* 自定义一个pojo

```java
@Data
public class SecurityUser implements UserDetails {
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    public String getPassword() {
        return new BCryptPasswordEncoder().encode("123456");
    }

    @Override
    public String getUsername() {
        return "ledger";
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

* 新建一个UserDetailsService实现类(获取用户的身份信息，会走数据库)

```java
@Service
public class UserServiceImpl implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        if(username==null || username.equals("")){
            throw new UsernameNotFoundException("用户名为空");
        }
        if(!username.equals("ledger")){
            throw new UsernameNotFoundException("用户名不存在");
        }
        return new SecurityUser();
    }
}
```

## 基于数据库的认证

* 基本原理，就是实现UserDetailsService，在这个类中根据用户名查询用户信息
  * 用户为null，就直接返回错误
  * 用户不为null就封装成UserDetails的实现，最后返回
  * 最后框架内部自己判断密码的正确性(要向容器中放入一个解密器)

* 数据库表映射pojo

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class SysUser {
    private Integer userId;
    private String username;
    private String password;
    private String sex;
    private String address;
    private Integer enabled;
    private Integer accountNoExpired;
    private Integer credentialsNoExpired;
    private Integer accountNoLocked;
    
}
```

* userdetails的实现类

```java
public class SecurityUser implements UserDetails {
    private final SysUser SYS_USER;

    public SecurityUser(SysUser SYS_USER){
        this.SYS_USER = SYS_USER;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    public String getPassword() {
        return SYS_USER.getPassword();
    }

    @Override
    public String getUsername() {
        return SYS_USER.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return SYS_USER.getAccountNoExpired() == 1;
    }

    @Override
    public boolean isAccountNonLocked() {
        return SYS_USER.getAccountNoLocked() == 1;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return SYS_USER.getCredentialsNoExpired() == 1;
    }

    @Override
    public boolean isEnabled() {
        return SYS_USER.getEnabled() == 1;
    }
}
```

* UserDetailsService的实现类

```java
@Service
public class SecurityUserDetailService implements UserDetailsService {
    @Resource
    private SysUserService sysUserService;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        SysUser sysUser = sysUserService.getUserByName(username);
        if(sysUser==null){
            throw new UsernameNotFoundException("用户名没有找到");
        }
        return new SecurityUser(sysUser);
    }
    
}
```

## 基于数据库的授权学习

* 根据用户id查询数据库获取用户的权限

```xml
  <select id="queryPermissionByUserId" resultType="java.lang.String">
    select distinct code
    from sys_role_user ru inner join sys_role_menu rm
    on  ru.rid=rm.rid inner join sys_menu  m
    on  rm.mid = m.id where ru.uid=1;
  </select>
```

* 用户安全类增加权限的set方法

```java
public class SecurityUser implements UserDetails {
    private final SysUser SYS_USER;

    public SecurityUser(SysUser SYS_USER){
        this.SYS_USER = SYS_USER;
    }
    //用于存储权限的接口
    private List<SimpleGrantedAuthority> authorities;

    public void setAuthorities(List<SimpleGrantedAuthority> authorities) {
        this.authorities = authorities;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public String getPassword() {
        return SYS_USER.getPassword();
    }

    @Override
    public String getUsername() {
        return SYS_USER.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return SYS_USER.getAccountNoExpired() == 1;
    }

    @Override
    public boolean isAccountNonLocked() {
        return SYS_USER.getAccountNoLocked() == 1;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return SYS_USER.getCredentialsNoExpired() == 1;
    }

    @Override
    public boolean isEnabled() {
        return SYS_USER.getEnabled() == 1;
    }
}

```

* 在userdetailservice的实现类里面，增加一个查到用户信息之后查询权限的方法

```java
@Service
public class SecurityUserDetailServiceImpl implements UserDetailsService {
    @Resource
    private SysUserService sysUserService;
    @Resource
    private SysMenuService sysMenuService;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        SysUser sysUser = sysUserService.getUserByName(username);
        if(sysUser==null){
            throw new UsernameNotFoundException("用户名没有找到");
        }
        //根据用户id获取用户的权限
        List<String> auths = sysMenuService.queryPermissionByUserId(sysUser.getUserId());
        SecurityUser securityUser = new SecurityUser(sysUser);
        List<SimpleGrantedAuthority> collect = auths
                .stream()
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());
        securityUser.setAuthorities(collect);
        return securityUser;
    }
}

```

## 集成图片验证码

* 使用糊涂包工具生成验证码图片
* 设置一个验证码的controller（将实际验证码放在session中）
* 配置拦截器，这个拦截器只会拦截验证码的路径，路径不走身份认证

```java
@Controller
@Slf4j
public class CaptchaController {
    @GetMapping("/getCaptchaCode")
    public void getCaptchaCode(HttpServletRequest request,HttpServletResponse response) throws IOException {
        CircleCaptcha circleCaptcha = CaptchaUtil.createCircleCaptcha(120, 60, 4, 600);
        String code = circleCaptcha.getCode();
        log.error(code);
        request.getSession().setAttribute("code",code);
        response.setContentType("image/jpeg");
        ImageIO.write(circleCaptcha.getImage(),"jpeg",response.getOutputStream());
    }
}
```

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307091945908.png)

```java
@Component
@Slf4j
public class ValidateCodeFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        String requestURI = request.getRequestURI();

        if (requestURI.equals("/getCaptchaCode")) {
            validateCode(request, response, filterChain);
        } else {
            filterChain.doFilter(request, response);
        }
    }

    private void validateCode(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws IOException, ServletException {
        HttpSession session = request.getSession();
        String code = (String) session.getAttribute("code");
        String code1 = request.getParameter("code");
        session.removeAttribute("code_err");
        if(code1==null || "".equals(code1)) {
            session.setAttribute("code_err","验证码为空");
        }
        if(!code.equals(code1)) {
            session.setAttribute("code_err","验证码");
        }
        PrintWriter writer = response.getWriter();
        writer.write(JSON.toJSONString(HttpResult.builder().code(400).msg((String) session.getAttribute("code_err")).build()));
        if (code.equals(code1)) {
            session.removeAttribute("code");
            filterChain.doFilter(request, response);
        }
    }
}

```

## JWT

* 三分组成
  1. 头部：加密算法类型、加密类型(jwt)，使用base64url编码。
  2. 负载：用户信息的加密，使用base64url编码。
  3. 签名：通过指定算法，通过Header和Playload加盐计算的字符串

### 生成token

```java
    public String createJwt(Integer userId, String userName, List<String> authList) {
        HashMap<String, Object> headerClaims = new HashMap<>();
        headerClaims.put("alg", "HS256");
        headerClaims.put("typ", "JWT");
        return JWT.create().withHeader(headerClaims)
                .withIssuer("ledger")//设置签发人
                .withIssuedAt(new Date())//设置签发时间
                .withExpiresAt(new Date(System.currentTimeMillis() + 60 * 60 * 1000 * 2))//设置过期时间2小时
                .withClaim("userId",userId)  //自定义声明
                .withClaim("userName",userName)  //自定义声明
                .withClaim("authList",authList)  //自定义声明
                .sign(Algorithm.HMAC256(secret));  //签名
    }
```

### jwt功能类

### 生成jwt

```java
    /**
     * 根据用户信息生成token
     *
     * @param userInfo   用户信息(securityUser的json形式)
     * @param authList 用户权限
     * @return jwt
     */
    public String createJwt(String userInfo, List<String> authList) {
        HashMap<String, Object> headerClaims = new HashMap<>();
        headerClaims.put("alg", "HS256");
        headerClaims.put("typ", "JWT");
        return JWT.create().withHeader(headerClaims)
                .withIssuer("ledger")//设置签发人
                .withIssuedAt(new Date())//设置签发时间
                .withExpiresAt(new Date(System.currentTimeMillis() + 30 * 60 * 1000))//设置过期时间2小时
                .withClaim("userInfo", userInfo)  //自定义声明
                .withClaim("authList", authList)  //自定义声明
                .sign(Algorithm.HMAC256(secret));  //签名
    }
```

### 校验jwt

```java
    /**
     * 校验token的正确性
     * @param jwtToken
     * @return
     */
    public Boolean verifyJwt(String jwtToken) {
        try {
            //创建一个jwt的校验器
            JWTVerifier require = JWT.require(Algorithm.HMAC256(secret)).build();
            //校验token（不正确会抛出异常）
            DecodedJWT verify = require.verify(jwtToken)
            return true;
        } catch (Exception e) {
            log.error("jwt不正确");
            return false;
        }
    }
```

### 在json里面获取荷载(用户信息和权限信息)

```java
    /**
     * 从token中获取用户info
     *
     * @param jwtToken
     * @return 用户info
     */
    public String getUserInfoFromToken(String jwtToken) {
        JWTVerifier require = JWT.require(Algorithm.HMAC256(secret)).build();
        //校验token
        DecodedJWT verify = require.verify(jwtToken);
        return verify.getClaim("userInfo").asString();
    }


      /**
     * 获取用户权限信息
     * @param jwtToken
     * @return
     */
    public List<String> getAuthListFromToken(String jwtToken) {
        JWTVerifier require = JWT.require(Algorithm.HMAC256(secret)).build();
        //校验token
        DecodedJWT verify = require.verify(jwtToken);
        List<SimpleGrantedAuthority> simpleGrantedAuthorityList = verify
                .getClaim("authList")
                .asList(SimpleGrantedAuthority.class);
        return simpleGrantedAuthorityList
                .stream()
                .map(SimpleGrantedAuthority::getAuthority)
                .collect(Collectors.toList());
    }
```

## 使用redis优化退出校验token

1. 引入redis依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

```yml
  redis:
    host: 111111
```

2. 在登录成功的时候，将jwt和用户信息写入redis(ttl的时长和jwt过期时长一致)

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307101433992.png)

3. 在退出登录的时候，将redis里的数据移出

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307101434230.png)

4. 在jwtCheck的过滤器校验用户的token是不是过期(加入redis的校验)

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307101435132.png)



