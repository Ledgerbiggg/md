# springsecurity的使用

## 后端
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
                .disable()
                //禁用session 的记住用户的功能
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);

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
        return Result.success("登录成功");
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
* 登录页面
```html
  <div class="body">
    <div class="loginBox">
      <h2>login</h2>
      <div>
        <div class="item">
          <input type="text" required v-model="user.username">
          <label for="">userName</label>
        </div>
        <div class="item">
          <input type="password" required v-model="user.password">
          <label for="">password</label>
        </div>
        <div class="item">
          <input type="text" required v-model="user.captcha">
          <label for="">captcha</label>
        </div>
        <img class="captcha" src="/api/captcha" @click="getCaptcha" ref="captcha">
        <button class="btn" @click="submit">submit
          <span></span>
          <span></span>
          <span></span>
          <span></span>
        </button>
      </div>
    </div>
  </div>
```
```js
import http from "@/js/http";

export default {
  name: "Login",
  data() {
    return {
      user: {
        username: "",
        password: "",
        captcha: ""
      },
    }
  },
  mounted() {
    this.getCaptcha()
  },
  methods: {
    getCaptcha() {
        //获取验证码(image流直接渲染,配置了代理,直接向本地请求)
      this.$refs.captcha.src = '/api/captcha?' + Math.random()
    },
    submit() {
      http.post(`/login?code=${this.user.captcha}`, this.user).then(res => {
        console.log("/login", res)
        if (res.data.code === 200) {
            //登录成功就跳转
          this.$router.push("/main")
        }
      }).catch(rea => {
        console.error("rea.data", rea.data)
        this.getCaptcha()
      })
    }
  }
}
```
* vue.config
```js
const { defineConfig } = require('@vue/cli-service')

module.exports = defineConfig({
  devServer: {
    proxy: {
      '/api': {
        target: 'http://localhost:9999', // 设置你的后端服务器地址
        changeOrigin: true, // 允许跨域
        pathRewrite: {
          '^/api': '/' // 如果后端接口没有以 '/api' 开头，可以去掉这一行
        }
      }
    }
  },
  transpileDependencies: true,
  lintOnSave: false
})

```
* 响应拦截器
```js

// 3.响应拦截器
****   request.js   ****/
// 导入axios
import axios from 'axios'
// 使用element-ui Message做消息提醒
import {Message} from 'element-ui';
// import router from "@/js/router";

//1. 创建新的axios实例，
const service = axios.create({
    baseURL: "http://localhost:8080/api",
    // baseURL: "http://ledger-code.buzz:9999",
    // baseURL: "http://ledgerhhh-ai.top:8080",
    // baseURL: "http://ledgerhhh-ai.top:8080",
    // 超时时间 单位是ms，这里设置了3s的超时时间
    timeout: 9 * 10000,
})
// 2.请求拦截器
service.interceptors.request.use(config => {

    //发请求前做的一些处理，数据转化，配置请求头，设置token,设置loading等，根据需求去添加
    config.data = JSON.stringify(config.data); //数据转化,也可以使用qs转换
    config.headers = {
        'Content-Type': 'application/json' //配置请求头
    }
    //如有需要：注意使用token的时候需要引入cookie方法或者用本地localStorage等方法，推荐js-cookie
    //const token = getCookie('名称');//这里取token之前，你肯定需要先拿到token,存一下
    // if(token){
    // config.params = {'token':token} //如果要求携带在参数中
    // config.headers.token= token; //如果要求携带在请求头中
    // }
    const token = window.localStorage.getItem('token');
    if (token) {
        // config.params = {'token':token} //如果要求携带在参数中
        config.headers.Authorization = token;
    }
    return config
}, error => {
    Promise.reject(error)
})

// 3.响应拦截器
service.interceptors.response.use(response => {
    //接收到响应数据并成功后的一些共有的处理，关闭loading等
    let token = response.headers.get('Authorization');
    if (token) {
        //获取token,刷新unexpired过期时间
        window.localStorage.setItem('token', token);
        window.localStorage.setItem('unexpired', 'ledger');
    }
    if (response.data.code === 403 ) {
        //未登录就去掉刷新
        window.localStorage.removeItem("unexpired")
        Message.error(response.data.msg)
        return Promise.reject(response);
    }
    if(response.data.code === 401){
        //没有权限
        Message.error(response.data.msg)
        return Promise.reject(response);
    }
    return response
}, error => {
    /***** 接收到异常响应的处理开始 *****/
    if (error && error.response) {
        // 1.公共错误处理
        // 2.根据响应码具体处理
        switch (error.response.status) {
            case 400:
                error.message = '错误请求'
                break;
            case 401:
                error.message = '未授权，请重新登录'
                break;
            case 403:
                error.message = '拒绝访问'
                break;
            case 404:
                error.message = '请求错误,未找到该资源'
                // window.location.href = "/NotFound"
                break;
            case 405:
                error.message = '请求方法未允许'
                break;
            case 408:
                error.message = '请求超时'
                break;
            case 500:
                error.message = '服务器端出错'
                break;
            case 501:
                error.message = '网络未实现'
                break;
            case 502:
                error.message = '网络错误'
                break;
            case 503:
                error.message = '服务不可用'
                break;
            case 504:
                error.message = '网络超时'
                break;
            case 505:
                error.message = 'http版本不支持该请求'
                break;
            default:
                error.message = `连接错误${error.response.status}`
        }
    } else {
        // 超时处理
        if (JSON.stringify(error).includes('timeout')) {
            Message.error('服务器响应超时，请刷新当前页')
        }
        error.message = '连接服务器失败'
    }

    Message.error(error.message)
    /***** 处理结束 *****/
    //如果不需要错误处理，以上的处理过程都可省略
    return Promise.resolve(error.response)
})
//4.导入文件
export default service
```
* router的路由守卫
```js
// 为路由对象，添加beforeEach导航守卫
router.beforeEach((to, from, next) => {
    //如果用户访问的登录页，直接放行
    if (to.path === '/login') return next()
    // 从sessionStorage中获取到保存的token值
    const tokenStr = window.localStorage.getItem('token')
    let unexpired = window.localStorage.getItem("unexpired");
    // 有token，而且未过期
    if(to.path!=='/main'){
        return next('/login')
    }
    if (tokenStr && unexpired) return next()
    if(tokenStr==null){
        Message.error('请先登录')
    }else {
        if (!unexpired){
            Message.error('身份认证过期')
        }
    }
    next('/login')
})
```
## 上线
* ngnix.conf
```conf
# 使用 "user" 指令来指定运行 Nginx 的用户，通常为 "nginx" 用户。
user nginx;

# "worker_processes" 指定了 Nginx 启动的工作进程数，"auto" 表示根据可用 CPU 核心数自动设置。
worker_processes auto;

# 定义错误日志的位置和文件名。
error_log /var/log/nginx/error.log;

# 配置事件模块，用于控制 Nginx 的事件处理。
events {
    # "worker_connections" 指定每个工作进程的最大并发连接数。
    worker_connections 1024;
}

# 配置 HTTP 模块，包含所有关于 HTTP 服务的设置。
http {
    # 包含 MIME 类型的配置文件，用于定义文件类型和对应的 MIME 类型。
    include /etc/nginx/mime.types;
    
    # 默认 MIME 类型，当无法确定文件类型时会使用该类型。
    default_type application/octet-stream;

    # 定义一个 HTTP 服务器块。
    server {
        # 监听端口为 80，用于处理传入的 HTTP 请求。
        listen 80;
        
        # 定义服务器名，通常是域名，可以是多个域名，用空格分隔。
        server_name ledger-code.buzz;

        # 配置请求的处理逻辑。
        location / {
            # 指定根目录，用于查找请求的资源文件。
            root /usr/share/nginx/html;
            
            # 默认索引文件名。
            index index.html;

            # 允许前端路由正常工作。
            try_files $uri $uri/ /index.html;
        }
        location /api/ {
            # 重写路径,去掉/api
            rewrite ^/api(/.*)$ $1 break;
            proxy_pass http://ledger-code.buzz:9999; # 后端服务器地址
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
         }  
    }
}

```
 * Dockerfile
```Dockerfile
# 使用一个基础的 Java 镜像
FROM openjdk:11-jre-slim

# 设置工作目录
WORKDIR /app

# 复制 JAR 文件到容器中
COPY english-1.0.jar /app/english-1.0.jar
COPY application.yml /app/application.yml

# 安装所需的字体包!!!!!(验证码里面配置的字体需要下载不然就会报错!!!!)
RUN apt-get update && apt-get install -y \
    fonts-wqy-zenhei \
    fonts-wqy-microhei \
    fonts-arphic-ukai \
    fonts-arphic-uming

# 暴露端口
EXPOSE 9999

# 定义启动命令
CMD ["java", "-jar", "english-1.0.jar"]
```
* docker-compose
```yml
version: '3'
services:
  # 启动nginx的镜像 docker run -p 80:80 \
  # -v ./nginx.conf:/etc/nginx/nginx.conf \
  # -v ./dist:/usr/share/nginx/html \
  # --name compose-nginx-1 nginx
  nginx:
    # 镜像名称
    image: nginx
    # 端口映射
    ports:
      - "80:80"
    # 挂载卷
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro  # 映射 NGINX 配置文件(ro表示容器内部只读,保证数据的安全性)
      - ./dist:/usr/share/nginx/html:ro # 映射 静态资源类dist 
    container_name: nginx

  # 打包java的镜像配置 docker build -t java-app .
  # 运行镜像的配置 docker run -p 9999:9999 \ 
  # -v ./application.yml:/app/application.yml \
  # --name compose-java-app-1 english
  java-app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "9999:9999"
    volumes:
      - ./application.yml:/app/application.yml:ro  # 映射 Java 应用程序配置文件
    container_name: english
```
* 启动
```sh
docker-compose up -d 
```


























































