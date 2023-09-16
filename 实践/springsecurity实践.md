## springsecurity的使用
1. 引入依赖
```xml
        <!-- 这个是security -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <!-- 这个是谷歌的验证码依赖 -->
        <dependency>
            <groupId>com.github.axet</groupId>
            <artifactId>kaptcha</artifactId>
            <version>0.0.9</version>
        </dependency>
```
2. WebSecurityConfig配置(新版使用@EnableWebSecurity这个注解,extends WebSecurityConfigurerAdapter已经被弃用)
```java
//定义这为Security的配置类
@EnableWebSecurity
//开启全局前置方法校验
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig {

    @Resource
    //权限不足处理器
    private AppAccessDeniedHandler appAccessDeniedHandler;

    @Resource
    //登录失败的重定向替代方案
    private AuthenticationEntryPoint authenticationEntryPoint;

    @Resource
    //jwt校验的过滤器
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;

    @Resource
    //验证码校验过滤器
    private ValidateCodeFilter validateCodeFilter;

    @Resource
    // 自定义的用户服务类
    private UserService userService;


    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
         httpSecurity
                // 禁用跨域资源共享 (CORS) 功能
                .cors().disable()
                // 在 UsernamePasswordAuthenticationFilter 之前添加自定义验证码过滤器
                .addFilterBefore(validateCodeFilter, UsernamePasswordAuthenticationFilter.class)
                // 在 UsernamePasswordAuthenticationFilter 之前添加JWT认证令牌过滤器
                .addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class)
                // 配置权限控制规则
                .authorizeRequests()
                // 任何请求都需要认证通过（需要登录）
                .anyRequest()
                .authenticated()
                .and()
                // 禁用跨站请求伪造 (CSRF) 保护
                .csrf().disable()
                // 配置会话管理策略
                .sessionManagement()
                // 会话创建策略为无状态，即不使用会话来存储用户状态
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                // 配置HTTP响应头部，包括缓存控制(禁用缓存)
                .headers()
                .disable();

        // 配置异常处理 - 设置身份验证入口点 (AuthenticationEntryPoint)
        httpSecurity
                .exceptionHandling()
                // 设置身份验证入口点，用于处理未经身份验证的请求(这个可以替代默认的重定向到/login!!!!!!!)
                .authenticationEntryPoint(authenticationEntryPoint);

        // 配置异常处理 - 设置访问被拒绝的处理程序 (AccessDeniedHandler)
        httpSecurity
                .exceptionHandling()
                // 设置访问被拒绝的处理程序，用于处理未经授权的请求
                .accessDeniedHandler(appAccessDeniedHandler);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        //加密器
        return new BCryptPasswordEncoder();
    }

    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        //不拦截的请求(只有UsernamePasswordAuthenticationFilter不去拦截这两个,但是validateCodeFilter和jwtAuthenticationTokenFilter会拦截)
        return web -> web.ignoring().antMatchers("/captcha", "/login");
    }

    @Bean
    public UserDetailsService userDetailsService() {
        //注入一个UserDetailsService进入
        return name -> new SecurityUser(userService.getUserByUsername(name));
    }

}
```
3. 挨个配置
* AccessDeniedHandler(权限不足处理器)
```java
@Component
@Slf4j
public class AppAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException)
            throws IOException, ServletException {
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        log.error("权限不足");
        writer.write(JSON.toJSONString(Result.fail("权限不足,请联系ledger获取管理员权限",401)));
        writer.flush();
        writer.close();
    }
}
```
* 用户登录不成功处理器
```java
@Component
@Slf4j
public class AppAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        writer.write(JSON.toJSONString(Result.fail("用户名或者密码不正确",403)));
        writer.flush();
        writer.close();
    }
}

```
* jwt过滤器
```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {
    @Resource
    private UserService userService;
    @Value("${jwt.tokenHeader}")
    private String tokenHeader;
    @Value("${jwt.tokenHead}")
    private String tokenHead;
    @Value("${jwt.secret}")
    private String secret;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        //不拦截登录请求
        if (request.getRequestURI().equals("/login")) {
            filterChain.doFilter(request, response);
            return;
        }

        // 从请求头中获取令牌
        String header = request.getHeader(tokenHeader);
        
        // 检查令牌是否存在且以指定的前缀开始
        if (header != null && header.startsWith(tokenHead)) {
            // 从令牌中验证令牌的有效性
            String token = header.replace(tokenHead + " ", "");
            if (JwtUtil.validateJwt(token, secret)) {
                // 从令牌中获取用户名
                String userNameFromToken = JwtUtil.getUserNameFromToken(token, secret);
                if (userNameFromToken != null) {
                    // 根据用户名获取用户信息
                    User userByUsername = userService.getUserByUsername(userNameFromToken);
                    
                    // 创建一个包含用户信息的SecurityUser对象
                    SecurityUser securityUser = new SecurityUser(userByUsername);
                    
                    // 创建一个身份验证令牌
                    UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken =
                            new UsernamePasswordAuthenticationToken(securityUser, null, securityUser.getAuthorities());
                    
                    // 将请求详情添加到身份验证令牌
                    usernamePasswordAuthenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    
                    // 将身份验证令牌设置到Security上下文
                    SecurityContextHolder.getContext().setAuthentication(usernamePasswordAuthenticationToken);
                    
                    // 继续处理请求
                    filterChain.doFilter(request, response);
                    
                    // 在响应头中添加新的令牌（刷新令牌）
                    response.setHeader(tokenHeader, tokenHead + " " + JwtUtil.refreshToken(token, secret));
                    return;
                }
            }
        }
        
        // 如果没有合法的令牌，则继续处理请求
        filterChain.doFilter(request, response);
    }
}

```
* jwt工具
```java
public class JwtUtil {
    public static final String CLAIM_KEY_USERNAME = "sub";
    public static final String CLAIM_KEY_CREATE = "create";
    public static final Integer EXPIRE_TIME = 20 * 60 * 1000;

    /**
     * 创建一个 JSON Web Token (JWT)。
     *
     * @param secret        用于签名JWT的密钥，确保令牌的完整性和真实性。
     * @return 生成的JWT字符串。
     */
    public static String createJwt(UserDetails userDetails, String secret) {
        String username = userDetails.getUsername();
        HashMap<String, Object> claims = new HashMap<>();
        claims.put(CLAIM_KEY_USERNAME, username);
        claims.put(CLAIM_KEY_CREATE, new Date());
        return createJwt(claims, secret,userDetails.getUsername());
    }

    private static String createJwt(Map<String, Object> claims, String secret, String username) {
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(username)
                .setExpiration(DateTime.now().plusSeconds(EXPIRE_TIME).toDate()) //设置过期时间
                .signWith(SignatureAlgorithm.HS512, secret) // 加密签名
                .compact();
    }

    /**
     * 校验jwt的生成的token
     *
     * @param jwtToken token
     * @param secret   密钥
     * @return true:校验成功;false:校验失败
     */

    public static boolean validateJwt(String jwtToken, String secret) {
        try {
            Claims claims = Jwts.parser()
                    .setSigningKey(secret)
                    .parseClaimsJws(jwtToken)
                    .getBody();
            // 在此处添加任何其他的验证逻辑，例如过期时间等
            Date expiration = claims.getExpiration();
            // JWT已过期
            if (expiration == null) {
                return false;
            }
            return !expiration.before(new Date());
        } catch (Exception e) {
            return false;
        }
    }


    /**
     * 从token中获取用户名
     * @param jwtToken token
     * @param secret 秘钥
     * @return 用户名
     */
    public static String getUserNameFromToken(String jwtToken, String secret) {
        Claims claimsFromToken = getClaimsFromToken(jwtToken, secret);
        return claimsFromToken.get(CLAIM_KEY_USERNAME, String.class);
    }
    public static String refreshToken(String jwtToken, String secret) {
        Claims claimsFromToken = getClaimsFromToken(jwtToken, secret);
        return createJwt(claimsFromToken, secret, claimsFromToken.getSubject());
    }

    /**
     * 从token中获取负载
     * @param jwtToken token
     * @param secret   秘钥
     * @return 荷载
     */
    private static Claims getClaimsFromToken(String jwtToken, String secret) {
        try {
            return Jwts.parser()
                    .setSigningKey(secret)
                    .parseClaimsJws(jwtToken)
                    .getBody();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }


}

```
* 验证码过滤
```java

@Slf4j
@Component
public class ValidateCodeFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        String requestURI = request.getRequestURI();
        if ("/login".equals(requestURI)) {
            validateCode(request, response, filterChain);
        } else {
            filterChain.doFilter(request, response);
        }
    }
    private void validateCode(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws IOException, ServletException {
        response.setContentType("application/json;charset=utf-8");
        HttpSession session = request.getSession();
        String code = (String) session.getAttribute("code");
        //验证码拿取一次就移除
        session.removeAttribute("code");
        String code1 = request.getParameter("code");
        session.removeAttribute("code_err");
        if (code1 == null || code == null) {
            session.setAttribute("code_err", "验证码为空");
        }
        if (!Objects.equals(code, code1)) {
            session.setAttribute("code_err", "验证码不正确");
        }
        if (!Objects.equals(code, code1) && code1 != null) {
            //如果验证码不一样就返回错误
            log.info("验证码不正确");
            PrintWriter writer = response.getWriter();
            writer.write(JSON.toJSONString(Result.fail(null,(String) session.getAttribute("code_err"),403)));
            writer.flush();
        }else {
            //进入login
            log.info("验证码正确");
            filterChain.doFilter(request, response);
        }

    }
}
```
* 验证码配置
```java

@Configuration
public class CaptchaConfig {
    @Bean
    public DefaultKaptcha defaultKaptcha() {

        //验证码生成器
        DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
        //配置
        Properties properties = new Properties();
        //是否有边框
        properties.setProperty("kaptcha.border", "yes");
        //设置边框颜色
        properties.setProperty("kaptcha.border.color", "105,179,90");
        //边框粗细度，默认为1
        // properties.setProperty("kaptcha.border.thickness","1");
        //验证码
        properties.setProperty("kaptcha.session.key", "code");
        //验证码文本字符颜色 默认为黑色
        properties.setProperty("kaptcha.textproducer.font.color", "blue");
        //设置字体样式
        properties.setProperty("kaptcha.textproducer.font.names", "宋体,楷体,微软雅黑");
        //字体大小，默认40
        properties.setProperty("kaptcha.textproducer.font.size", "30");
        //验证码文本字符内容范围 默认为abced2345678gfynmnpwx
        // properties.setProperty("kaptcha.textproducer.char.string", "");
        //字符长度，默认为5
        properties.setProperty("kaptcha.textproducer.char.length", "4");
        //字符间距 默认为2
        properties.setProperty("kaptcha.textproducer.char.space", "4");
        //验证码图片宽度 默认为200
        properties.setProperty("kaptcha.image.width", "100");
        //验证码图片高度 默认为40
        properties.setProperty("kaptcha.image.height", "40");
        Config config = new Config(properties);
        defaultKaptcha.setConfig(config);
        return defaultKaptcha;
    }
}

```
* 验证码控制层
```java

@RestController
@Slf4j
public class CaptchaController {

    @Resource
    private DefaultKaptcha defaultKaptcha;

    //produces配合swagger在图形化界面中看到图片而不是乱码
    @GetMapping(value = "/captcha",produces = "image/jpeg")
    public void captcha(HttpServletRequest request, HttpServletResponse response) {
        response.setContentType("image/jpeg");
        //获取文本内容
        String text = defaultKaptcha.createText();
        log.warn("验证码文本内容{}", text);
        //保存文本
        request.getSession().setAttribute("code", text);
        //使用文本创建图像
        BufferedImage image = defaultKaptcha.createImage(text);
        ServletOutputStream outputStream = null;
        try {
            outputStream = response.getOutputStream();
            //输出流输出图片
            ImageIO.write(image,"jpg",outputStream);
            //刷新缓存
            outputStream.flush();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                //关闭流
                outputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```
* 登录控制层
```java
@RestController
@Slf4j
public class LoginController {

    @Resource
    private UserService userService;

    @PostMapping("/login")
    public Result<String> login(@RequestBody User user, HttpServletResponse response) {
        return userService.login(user, response);
    }


}

```
* 登录服务
```java

@Service
@Slf4j
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {

    @Value("${jwt.secret}")
    private String secret;
    @Value("${jwt.tokenHead}")
    private String tokenHead;


    @Override
    public Result<String> login(User user, HttpServletResponse response) {
        String username = user.getUsername();
        User userByUsername = getUserByUsername(username);
        if (userByUsername == null) {
            return Result.fail("用户名不存在",403);
        }
        if (!new BCryptPasswordEncoder().matches(user.getPassword(), userByUsername.getPassword())) {
            return Result.fail("密码错误", 403);
        }
        SecurityUser securityUser = new SecurityUser(userByUsername);
        String token = JwtUtil.createJwt(securityUser, secret);
        response.setHeader("Authorization", tokenHead + " " + token);
        return Result.success("登录成功", token);
    }

    @Override
    public User getUserByUsername(String username) {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getUsername, username);
        return getOne(wrapper);
    }


}


```
* user类
```java
@Data
@TableName("users")
@AllArgsConstructor
@NoArgsConstructor
public class User {
    @TableId
    private String id;
    private String username;
    private String password;
    private String role;
    private LocalDateTime created_at;
    private LocalDateTime updated_at;

}

```
* 安全用户类
```java

@Data
public class SecurityUser implements UserDetails {
    private final User user;

    public SecurityUser(User user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        SimpleGrantedAuthority simpleGrantedAuthority = new SimpleGrantedAuthority(user.getRole());
        ArrayList<GrantedAuthority> grantedAuthorities = new ArrayList<>();
        grantedAuthorities.add(simpleGrantedAuthority);
        return grantedAuthorities;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
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

## 前端部分

































































