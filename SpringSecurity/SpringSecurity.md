
# 认证
## 实现逻辑
### 登录
  * ①自定义登录接口
    调用ProviderManager的方法进行认证 如果认证通过生成jwt
    把用户信息存入redis中
  * ②自定义UserDetailsService 
    在这个实现列中去查询数据库
### 校验
  * ①定义Jwt认证过滤器
    获取token
    解析token获取其中的userid
    从redis中获取用户信息
    存入SecurityContextHolder
## 实现
### User类对应数据库中的sys_user
```java
/**
 * 用户表(User)实体类
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@TableName(value = "sys_user")
public class User implements Serializable {
    private static final long serialVersionUID = -40356785423868312L;

    /**
     * 主键
     */
    @TableId
    private Long id;
    /**
     * 用户名
     */
    private String userName;
    /**
     * 昵称
     */
    private String nickName;
    /**
     * 密码
     */
    private String password;
    /**
     * 账号状态（0正常 1停用）
     */
    private String status;
    /**
     * 邮箱
     */
    private String email;
    /**
     * 手机号
     */
    private String phonenumber;
    /**
     * 用户性别（0男，1女，2未知）
     */
    private String sex;
    /**
     * 头像
     */
    private String avatar;
    /**
     * 用户类型（0管理员，1普通用户）
     */
    private String userType;
    /**
     * 创建人的用户id
     */
    private Long createBy;
    /**
     * 创建时间
     */
    private Date createTime;
    /**
     * 更新人
     */
    private Long updateBy;
    /**
     * 更新时间
     */
    private Date updateTime;
    /**
     * 删除标志（0代表未删除，1代表已删除）
     */
    private Integer delFlag;
}

```

### UserDetailService的实现类
该类实现根据username从数据库中查询用户完整信息, 返回UserDetail的实现类
```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //查询用户信息
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getUserName,username);
        User user = userMapper.selectOne(wrapper);
        //若查询结果为空 抛出异常
        if(Objects.isNull(user)) throw new RuntimeException("用户名错误");
        //将查询结果封装成UserDetails
        return new LoginUser(user);
    }
}
```

### UserDetail的是实现类
该类未UserDetailService接口的实现类的返回值
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class LoginUser implements UserDetails {

    @Autowired User user;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUserName();
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

### 功能实现类
该类实现登录和退出功能
* 登录
  1. 将查询到的user封装成Authencation接口的实现类UsernamePasswordAuthenticationToken
  2. 调用AuthenticationManager..authenticate()进行用户认证, 该方法返回一个Authentication对象
  3. 通过返回的Authentication对象获取LoginUser对象
  4. 将LoginUser对象封装成JWT, 再将JWT对象封装近放回类中
  5. 将完整用户信息存入redis

* 退出
  1. 从SecurityContextHolder获取Authentication对象
  2. 通过Authentication.getPrincipal()获取LoginUser对象
  3. 删除redis中的完整用户信息 
```java
@Service
public class LoginServiceImpl implements LoginService{
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private RedisCache redisCache;

    @Override
    public ResponseResult login(User user) {
        //把用户登录时的信息封装成authentication对象
        UsernamePasswordAuthenticationToken token =
                new UsernamePasswordAuthenticationToken(user.getUserName(),user.getPassword());
        //1. AuthenticationManage 进行用户认证
        Authentication authentication = authenticationManager.authenticate(token);
        //2. 如果没认证通过 给出对应提示
        if(Objects.isNull(authentication)){
            throw new RuntimeException("登录失败");
        }
        //3.如果认证通过了 使用userid生成jwt 装入ResponseResult返回
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();
        String userid = loginUser.getUser().getId().toString();
        String jwt = JwtUtil.createJWT(userid);
        //4. 把完整的用户信息装入redis userid作为key
        Map<String ,String> map = new HashMap<>();
        map.put("token",jwt);
        redisCache.setCacheObject("login:"+userid,loginUser);
        return new ResponseResult(200,"登录成功",map);
    }

    @Override
    public ResponseResult logout() {
        //获取SecurityContextHolder中的用户id
        UsernamePasswordAuthenticationToken authentication =
                (UsernamePasswordAuthenticationToken) SecurityContextHolder.getContext().getAuthentication();
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();
        //删除redis中的值
        Long userid = loginUser.getUser().getId();
        redisCache.deleteObject("login:"+userid);
        return new ResponseResult(200,"注销成功",null);
    }
}

```

### 自定义过滤器
过滤器实现:
  1. 获取请求头中的token
  2. 解析token获取userid
  3. 根据userid从redis中获取完整的用户信息
  4. 将用户信息存入SecurityContextHolder
  5. 放行

```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {
    @Autowired
    private RedisCache redisCache;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        //1. 获取token
        String token = request.getHeader("token");
        //放行
        if(!StringUtils.hasText(token)){
            filterChain.doFilter(request,response);
            return;
        }
        //2. 解析token
        String userid;
        try {
            Claims claims = JwtUtil.parseJWT(token);
            userid = claims.getSubject();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("token非法");
        }
        String redisKey = "login:"+userid;
        //3. 从redis获取用户信息
        LoginUser  loginUser = redisCache.getCacheObject(redisKey);
        //4. 存入SecurityContextHolder
        if(Objects.isNull(loginUser)){
            throw new RuntimeException("用户未登录");
        }
        //TODO 获取权限信息要封装到Authentication中
        UsernamePasswordAuthenticationToken passwordAuthenticationToken =
                new UsernamePasswordAuthenticationToken(loginUser,null,null);
        SecurityContextHolder.getContext().setAuthentication(passwordAuthenticationToken);
        //放行
        filterChain.doFilter(request,response);
    }
}
```

### 配置类
SpringSecurity的配置类需要继承WebSecurityConfigurerAdapter
需要配置:
1. 注入PasswordEncoder 加密方式
2. 注入AuthenticationManager 用于认证
3. 重写 void configure(HttpSecurity http方法):
    (1) 配置放行
    (2) 将自定义过滤器加入过滤链




```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;

    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //关闭csrf
                .csrf().disable()
                //不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 对于登录接口 允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();
        //将自定义过滤器添加到UsernamePasswordAuthenticationFilter之前
        http.addFilterBefore(jwtAuthenticationTokenFilter,UsernamePasswordAuthenticationFilter.class);
    }
}
```

# 授权
## 相关配置
```java
// 开启@PreAuthorize注解使用
@EnableGlobalMethodSecurity(prePostEnabled = true)
```
```java
//在控制器或控制器方法上使用注解设置访问资源的权限
 @PreAuthorize("hasAuthority('test')")
```

## 自定义权限校验方法
```java
/**
* 自定义权限检查类
**/
@Component
public class MySecurityRoot {
    public boolean hasAuthority(String authority){
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        LoginUser user = (LoginUser) authentication.getPrincipal();
        List<String> authorities = user.getAuthentication();
        return authorities.contains(authority);
    }
}
```
使用方法
```java
    @RequestMapping("/ex")
    @PreAuthorize("@myExpression.hasAuthority('test')")
    public String ex(){
        return "my expression";
    }
```

## 在UserDetail的实现类中添加权限
```java
public class LoginUser implements UserDetails {
    private User user;
    private List<String> authentication;
    //不支持序列化, 防止redis异常, 只需要存入原始权限集合
    @JSONField(serialize = false)
    private List<GrantedAuthority> authorities;

    //带权限列表的构造函数
    public LoginUser(User user, List<String> authentication) {
        this.user = user;
        this.authentication = authentication;
    }

    //返回权限信息的方法
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        if(authorities!=null)
            return authorities;
        authorities =  authentication.stream().map(SimpleGrantedAuthority::new).collect(Collectors.toList());
        return authorities;
	}
}
```

## 将权限封装到token中
```java
UsernamePasswordAuthenticationToken passwordAuthenticationToken =
                new UsernamePasswordAuthenticationToken(loginUser,null,loginUser.getAuthorities());
        SecurityContextHolder.getContext().setAuthentication(passwordAuthenticationToken);
```

# 自定义失败处理
在SpringSecurity中，如果我们在认证或者授权的过程中出现了异常会被ExceptionTranslationFilter捕获到。在ExceptionTranslationFilter中会去判断是认证失败还是授权失败出现的异常。
## 认证失败处理
认证异常会被封装成AuthenticationException然后调用**AuthenticationEntryPoint**对象的方法去进行异常处理。
```java
/**
 * 认证失败处理
 */
@Component
public class AuthenticationEntryPointImpl implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        //认证失败返回的数据
        ResponseResult result = new ResponseResult(HttpStatus.UNAUTHORIZED.value(), "用户名或密码错误",null);
        String json = JSON.toJSONString(result);
        WebUtils.renderString(response,json);
    }
}
```

## 授权失败处理
授权异常会被封装成AccessDeniedException然后调用**AccessDeniedHandler**对象的方法去进行异常处理。
```java
/**
 * 授权失败处理
 */
@Component
public class AccessDeniedHandlerImpl implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        ResponseResult result = new ResponseResult(HttpStatus.FORBIDDEN.value(), "无权限访问",null);
        String json = JSON.toJSONString(request);
        WebUtils.renderString(response,json);
    }
}
```

## 将异常处理器添加到配置
```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //配置异常处理器
        http.exceptionHandling()
                .authenticationEntryPoint(authenticationEntryPoint)
                .accessDeniedHandler(accessDeniedHandler);
    }
```

# 解决跨域问题
## springboot允许跨域请求
```java
/**
 * springboot 允许跨域配置
 */
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // 设置允许跨域的路径
        registry.addMapping("/**")
                // 设置允许跨域请求的域名
                .allowedOriginPatterns("*")
                // 是否允许cookie
                .allowCredentials(true)
                // 设置允许的请求方式
                .allowedMethods("GET", "POST", "DELETE", "PUT")
                // 设置允许的header属性
                .allowedHeaders("*")
                // 跨域允许时间
                .maxAge(3600);
    }
}
```

## springsecurity允许跨域
```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //允许跨域
        http.cors();
    }
```

# RABC基于角色的权限控制
