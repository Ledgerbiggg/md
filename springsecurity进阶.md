## 这个是针对用户的角色权限访问进行url的限制的springsecurity代码设计

### 数据库表的设计如下

* 根据用户所要访问的菜单(url路径) 匹配相应的角色信息(角色和菜单是多对多关系)

* menu

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307111621047.png)

* menu_role

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307111623322.png)

* role

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307111625353.png)

### 获取所有菜单的信息并且可以访问角色这个菜单信息

```sql
        select
            m.*,
            r.id as rid,
            r.name as rname,
            r.nameZh as rnameZh
        from
            t_menu m left join
            t_menu_role mr on m.id = mr.mid left join
            t_role r on r.id =mr.rid
        order by m.id;
```

### 对菜单访问的角色信息进行收集

```java
@Component
public class CustomFilter implements FilterInvocationSecurityMetadataSource {
    @Autowired
    private IMenuService menuService;

    private AntPathMatcher antPathMatcher = new AntPathMatcher();

    /**
     * 根据请求的URL获取相应的权限配置。
     * @param o 过滤器调用对象，包含了当前的请求信息。
     * @return 权限配置集合，表示该请求需要的角色权限。
     * @throws IllegalArgumentException 如果参数o不是FilterInvocation对象。
     */
    @Override
    public Collection<ConfigAttribute> getAttributes(Object o) throws IllegalArgumentException {
        // 获取请求的URL
        String requestUrl = ((FilterInvocation) o).getRequestUrl();
        // 获取所有菜单及其对应的角色信息
        List<Menu> menuWithRole = menuService.getMenuWithRole();
        // 遍历菜单，根据URL匹配查找角色配置
        for (Menu menu : menuWithRole) {
            if (antPathMatcher.match(menu.getUrl(), requestUrl)) {
                // 将菜单对应的角色信息转换为字符串数组
                String[] collect =
                    menu.getRoles().stream().map(Role::getName).toArray(String[]::new);
                // 返回权限配置集合，表示该请求需要的角色权限
                return SecurityConfig.createList(collect);
            }
        }
        // 如果没有匹配的菜单，返回包含"ROLE_LOGIN"角色的权限配置集合，表示需要登录才能访问该请求
        return SecurityConfig.createList("ROLE_LOGIN");
    }

    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        return null;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return false;
    }
}

```

这样就将传入的url和所要需要的角色信息存入，如果不需要用户的角色信息，就设置一个ROLE_LOGIN的身份，保证你是一个已经登录的角色的信息才可以访问

### 用户信息校验

1. 根据用户的token中的信息，用用户 id查询对应的角色(role)信息，再从数据库表对应关系中找到对应的菜单是否可以访问

```sql
        select
            r.id,
            r.name,
            r.nameZh
        from
            t_role r left join t_admin_role ar on r.id=ar.rid
        where ar.adminId=#{adminId};
```

* 这个语句是用来判断角色身份信息的

2. 根据用户的身份信息存入userdetails的实现类
   * 我是这样实现的：
     1. 在adminserviceimpl中写一个根据用户名查询用户以及根据用户id查询角色的方法因为我的admin已经实现了userdetails这个接口，并且也已经增加了角色属性，我将角色和权限进行关联
   * 这个是admin的pojo

```java
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
@TableName("t_admin")
@ApiModel(value="Admin对象", description="")
public class Admin implements Serializable, UserDetails {

    private static final long serialVersionUID = 1L;

    @ApiModelProperty(value = "id")
    @TableId(value = "id", type = IdType.AUTO)
    private Integer id;

    @ApiModelProperty(value = "姓名")
    private String name;

    @ApiModelProperty(value = "手机号码")
    private String phone;

    @ApiModelProperty(value = "住宅电话")
    private String telephone;

    @ApiModelProperty(value = "联系地址")
    private String address;

    @ApiModelProperty(value = "是否启用")
    private Boolean enabled;

    @ApiModelProperty(value = "用户名")
    private String username;

    @ApiModelProperty(value = "密码")
    private String password;

    @ApiModelProperty(value = "用户头像")
    private String userFace;

    @ApiModelProperty(value = "备注")
    private String remark;

    @ApiModelProperty(value = "角色")
    @TableField(exist = false)
    private List<Role> roles;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return roles.stream()
                .map(role -> new SimpleGrantedAuthority(role.getName()))
                .collect(Collectors.toList());
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
        return enabled;
    }
}

```

* 这个是adminserviceimpl的查询方法

```java
    @Override
    public Admin getAdminByUserName(String name) {
        LambdaQueryWrapper<Admin> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(Admin::getUsername, name)
                .eq(Admin::getEnabled, 1);
        Admin admin = getOne(wrapper);
        List<Role> rolesByAdminId = getRolesByAdminId(admin.getId());
        admin.setRoles(rolesByAdminId);
        return admin;
    }
```

* userdetailservice的实现类我是直接调用这个服务查询的admin(匿名内部类重写loaduserbyusername的方法)

```java
    @Override
    @Bean
    public UserDetailsService userDetailsService() {
        //重写用户信息获取的方法(根据用户民来走数据库，返回的对象是一个Admin，admin是实现了UserDetails的)
        return name -> adminService.getAdminByUserName(name);
    }

```

### 校验角色

* 之后就是校验这个角色有没有权限访问这个路径了

```java
/**
 * 权限控制判断用户角色
 *
 * @author ledger
 * @version 1.0
 **/
@Component
@Slf4j
public class CustomUrlDecisionManager implements AccessDecisionManager {
    /**
     * 根据权限配置决定是否允许用户访问资源。
     * @param authentication 当前用户的身份认证信息。
     * @param o 被保护的资源对象。
     * @param collection 包含了当前请求需要的角色权限配置。
     * @throws AccessDeniedException 如果用户没有足够的权限访问资源。
     * @throws InsufficientAuthenticationException 如果用户没有进行身份认证。
     */
    @Override
    public void decide(Authentication authentication, Object o, Collection<ConfigAttribute> collection)
            throws AccessDeniedException, InsufficientAuthenticationException {
        for (ConfigAttribute configAttribute : collection) {
            // 获取当前URL需要的角色
            String needRole = configAttribute.getAttribute();

            // 判断角色是否为登录即可访问角色，这个角色在CustomFilter中设置
            if ("ROLE_LOGIN".equals(needRole)) {
                // 如果用户没有进行身份认证，则抛出AccessDeniedException异常，要求用户登录
                if (authentication instanceof AnonymousAuthenticationToken) {
                    throw new AccessDeniedException("尚未登录，请登录");
                } else {
                    // 用户已经进行了身份认证，允许访问该资源
                    return;
                }
            }

            // 判断用户角色是否满足当前URL所需的角色权限
            Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
            for (GrantedAuthority authority : authorities) {
                if (authority.getAuthority().equals(needRole)) {
                    // 用户拥有所需角色权限，允许访问该资源
                    return;
                }
            }
        }

        // 用户权限不足，抛出AccessDeniedException异常
        throw new AccessDeniedException("权限不足");
    }


    @Override
    public boolean supports(ConfigAttribute configAttribute) {
        return false;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return false;
    }
}

```

### 对了不要忘记注册哦

* 注册动态权限匹配

```java
  @Override
    protected void configure(HttpSecurity http) throws Exception {
        //使用jwt不需要csrf
        http.csrf()
                .disable()
                //不需要session
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                //所有请求都要认证
                .anyRequest()
                .authenticated()
                //动态权限的配置
                .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
                    @Override
                    public <O extends FilterSecurityInterceptor> O postProcess(O o) {
                        //设置元数据
                        o.setSecurityMetadataSource(customFilter);
                        //设置元数据校验规则
                        o.setAccessDecisionManager(customUrlDecisionManager);
                        return o;
                    }
                })
                .and()
                //不使用缓存
                .headers()
                .cacheControl();
        //添加jwt登录授权过滤器
        http.addFilterBefore(jwtAuthencationTokenFilter(), UsernamePasswordAuthenticationFilter.class);
        //添加自定义未授权的结果返回
        http.exceptionHandling()
                .accessDeniedHandler(restfulAccessDeniedHandler)
                .authenticationEntryPoint(restAuthorizationEntryPoint);
    }
```

### 结果展示

* 可以访问的资源路径

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307111710102.png)

* 被拦截的资源路径

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307111710780.png)



### 作者提示

* 其实还可以用其他方法：
  * @EnableGlobalMethodSecurity(prePostEnabled = true)注解开启方法的url的权限限制(要给每个controller里面的方法都要增加注解@PreAuthorize("hasAuthority('sys-msg:add')"))、
  * 在configure(HttpSecurity http)这个重写的方法中直接配置等等(不过比较麻烦，写的东西比较多)，
* 控制权限访问的方法还有很多，自己可以尝试写写哦





















