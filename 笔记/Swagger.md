## swagger的简单配置

1. 引入依赖
   * 使用swagger-bootstrap-ui之后界面会更加好看和整洁

```xml
        <!-- swagger2 依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.7.0</version>
        </dependency>
        <!-- Swagger第三方ui依赖 -->
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>swagger-bootstrap-ui</artifactId>
            <version>1.9.6</version>
        </dependency>
```

2. 配置swagger

```java
@Configuration
@EnableSwagger2
public class Swagger2Config {
    /**
     * 扫描哪些包下面
     * @return
     */
    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.ledger.server.controller"))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo(){
        return new ApiInfoBuilder()
                .title("yeb")
                .description("接口文档")
                .contact(new Contact("ledger","22***584@qq.com",""))
                .version("1.2")
                .build();
    }
}
```

3. 配置springsecurity忽略的路径请求 （WebSecurityConfigurerAdapter的实现）

```java
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers(
                "/login",
                "/logout",
                "/css/**",
                "/js/**",
                "/images/**",
                "/index.html",
                "favicon.ico",
                "/doc.html",
                "/webjars/**",
                "/swagger-resources/**",
                "/v2/api-docs/**"
        );
    }
```

4. 访问http://localhost:8080/doc.html

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307101712427.png)

## swagger的默认auth的请求头配置

* 代码如下

```java
@Configuration
@EnableSwagger2
public class Swagger2Config {
    /**
     * 扫描哪些包下面
     *
     * @return
     */
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.ledger.server.controller"))
                .paths(PathSelectors.any())
                .build()
                .securityContexts(securityContexts())
                .securitySchemes(securitySchemes());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("yeb")
                .description("接口文档")
                .contact(new Contact("ledger", "228664584@qq.com", ""))
                .version("1.2")
                .build();
    }

    private List<ApiKey> securitySchemes() {
        //设置请求头信息
        List<ApiKey> result = new ArrayList<>();
        ApiKey apiKey = new ApiKey("Authorization", "Authorization", "header");
        result.add(apiKey);
        return result;
    }
    private List<SecurityContext> securityContexts(){
        ArrayList<SecurityContext> securityContexts = new ArrayList<>();
        securityContexts.add(getContextByPath("/hello/.*"));
        return securityContexts;
    }
    private SecurityContext getContextByPath(String pathRegex){
        return SecurityContext
                .builder()
                .securityReferences(defaultAuth())
                .forPaths(PathSelectors.regex(pathRegex)).build();
    }

    private List<SecurityReference> defaultAuth() {
        List<SecurityReference> result=new ArrayList<>();
        AuthorizationScope authorizationScope=new AuthorizationScope("global","accessEverything");
        AuthorizationScope[] authorizationScopes=new AuthorizationScope[1];
        authorizationScopes[0]=authorizationScope;
        result.add(new SecurityReference("Authorization",authorizationScopes));
        return result;
    }
}
```

* 界面新增配置

![](https://ledger-image-bed-1318365547.cos.ap-shanghai.myqcloud.com//image-bed202307101731107.png)

* 之后每次请求都会带一个auth的身份信息













