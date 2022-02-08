# SpringMVC简介

## 什么是MVC
MVC是一种软件架构的思想，将软件按照模型、视图、控制器来划分
M: Model，模型层，指工程中的JavaBean，作用是处理数据
JavaBean分为两类:
* —类称为实体类Bean:专门存储业务数据的，如Student user等
* 一类称为业务处理Bean:指Service或Dao对象，专门用于处理业务逻辑和数据访问。

V: View，视图层，指工程中的html或jsp等页面，作用是与用户进行交互，展示数据
C: Controller，控制层，指工程中的servlet，作用是接收请求和响应浏览器
MVC的工作流程:
用户通过视图层发送请求到服务器，在服务器中请求被Controller接收，Controller调用相应的Model层处理请求，处理完毕将结果返回到Controller，Controller再根据请求处理的结果找到相应的View视图，渲染数据后最终响应给浏览器

## 什么是SpringMVC
SpringMVC是Spring的一个后续产品，是Spring的一个子项目
SpringMVC是 Spring为表述层开发提供的一整套完备的解决方案。在**表述层**框架历经Strust、WebWork,Strust2等诸多产品的历代更迭之后，目前业界普遍选择了SpringMVC作为JavaEE项目表述层开发的首选方案。
>注:三层架构分为表述层（或表示层)、业务逻辑层、数据访问层，表述层表示前台页面和后台servlet

# 第一个SpringMVC程序
## 环境搭建
IDE : idea2021
构建工具 : maven
服务器 : tomcat
Spring版本 : 5以上

## 依赖
使用maven配置
	spring-webmvc
	logback-classic
	javax.servlet-api
	thymeleaf-spring5
## 配置web.xml
注册SpringMVC前端控制器DiapatcherServlet
### 默认配置方式
此配置作用下，SpringMVC的配置文件默认位于WEB-INF下，默认名称为<servlet-name>-servlet.xml
```xml
<!--配置前端控制器,对浏览器发送的请求进行统一处理-->
    <servlet>
        <servlet-name>SpringMVC01</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>SpringMVC01</servlet-name>
        <!--设置核心控制器所能处理的请求的路径
        / 匹配的请求式除了.jsp以外所有的请求
        /* 是所有请求
        -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```
### 扩展配置方式
可通过init-param标签设置SpringMVC配置文件的位置和名称，通过load-on-startup标签设置SpringMVC前端控制器DispatcherServlet的初始化时间
```xml
<servlet>
        <servlet-name>SpringMVC01</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--配置springMVC配置文件的位置和名称
        classpath : 类路径 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:SpringMVC01.xml</param-value>
        </init-param>
        <!--将前端控制器的初始化时间提前到服务器启动时-->
        <load-on-startup>1</load-on-startup>
    </servlet>
```
## 创建控制器
1. 使用注解@Controlle创建控制器类
2. 在spring配置文件中开启组件扫描

## 配置SpringMVC.xml配置文件
1. 开启组件扫描
2. 配置thymeleaf视图解析器

## 访问首页

```java
@Controller
public class HelloController {
    //请求映射
    @RequestMapping(value = "/")
    public String index(){
        //返回视图名称
        //根据在配置视图解析器中的视图前缀和视图后缀
        return "index";
    }
    @RequestMapping("/target")
    public String target(){
        return "target";
    }
}
```
## 跳转
这里使用th:标签 + thymeleaf语法
`<a th:href="@{/target}">target</a>`


## SpringMVC执行流程

1. 浏览器发送请求，若请求地址符合前端控制器的url-pattern，该请求就会被前端控制器DispatcherServlet处理。
2. 前端控制器会读取SpringMVC的核心配置文件，通过扫描组件找到控制器，将请求地址和控制器中
3. @RequestMapping注解的value属性值进行匹配，若匹配成功，该注解所标识的控制器方法就是处理请求的方法。处理请求的方法需要返回一个字符串类型的视图名称，该视图名称会被视图解析器解析，加上前缀和后缀组成视图的路径，通过Thymeleaf对视图进行渲染，最终转发到视图所对应页面.

# RequestMapping注解
## 功能
在Spring MVC 中使用 @RequestMapping 来映射请求，也就是通过它来指定控制器可以处理哪些URL请求，相当于Servlet中在web.xml中配置

## 注解位置
类定义处：规定初步的请求映射，相对于web应用的根目录；
方法定义处：进一步细分请求映射，相对于类定义处的URL。如果类定义处没有使用该注解，则方法标记的URL相对于根目录而言；
(意义类似于分级目录,允许不同目录下同名url)

## value属性
* value属性通过请求的请求地址匹配请求映射
* value属性是一个字符串类型的数组，表示该请求映射能够匹配多个请
`@RequestMapping(value = {"test01","test001"})`
* 求地址所对应的请求value属性必须设置，至少通过请求地址匹配请求映射

## method属性
* method属性通过请求的请求方式(get或post)匹配请求映射
* method属性是一个RequestMethod类型的数组，表示该请求映射能够匹配多种请求方式的请求
`method ={RequestMethod.GET,RequestMethod.POST}`
```java
public enum RequestMethod {
    GET,
    HEAD,
    POST,
    PUT,
    PATCH,
    DELETE,
    OPTIONS,
    TRACE;
    private RequestMethod() {
    }
}
```
* 若当前请求的请求地址满足请求映射的value属性，但是请求方式不满足method属性，则浏览器报错405:Request method 'POST'not supported

* 对于处理指定请求方式的控制器方法，SpringMVC中提供了@RequestMapping的派生注解
  处理get请求的映射->@GetMapping
  处理post请求的映射->@PostMapping
  处理put请求的映射-->@PutMapping
  处理delete请求的映射-->@DeleteMapping
  
* 浏览器只支持get和post请求,若在表单提交时设置了其他请求方式,则浏览器默认按get处理.若要使用put和delete请求则需要使用 spring提供的过滤器HiddenHttpMethodFilter.

* 请求方式错误报错405

## params属性

* params属性通过请求的请求参数匹配请求映射
* params属性是一个字符串类型的数组，可以通过四种表达式设置请求参数和请求映射的匹配关系
    "param":要求请求映射所匹配的请求必须携带param请求参数
    "!param":要求请求映射所匹配的请求必须不能携带param请求参数
    "param=value":要求请求映射所匹配的请求必须携带param请求参数并且值=value
    "param!=value":要求请求映射所匹配的请求必须不能携带param请求参数,并且值不等于value
* params 属性错误 报错400

## headers属性
用法与params一致,请求错误报错为404

## ant风格路径
? : 表示任意单个字符
\* : 表示任意0个或多个字符
\** : 表示 任意一层或多层目录
注意: \** 只能以 /**/  方式使用

## 路径占位符

@PathVariable注解,建立占位符和控制器方法形参的映射关系

```Java
    @RequestMapping("/test01/{id}/{username}")
    public String test01(@PathVariable("id")int id,@PathVariable("username")String username){
        System.out.println(id+" "+username);
        return "test01";
    }
```

# 获取请求参数

## 通过ServletAPI获取(一般不用)
MVC会根据形参列表自动装填请求对象
```Java
    @RequestMapping("/test01")
    public String test02(HttpServletRequest request){
        System.out.println(request.getParameter("id"));
        System.out.println(request.getParameter("username"));
        return "test01";
    }
```

## 通过控制器方法形参获取请求参数
* 方法形参名与属性名相同,MVC自动装填
* 若请求中包含多个同名请求参数,可以在控制器方法中使用字符串类型或字符串数组接收此参数
```Java
    @RequestMapping("test01test2")
    public String test03(String id,String username,String[] hobby){
        System.out.println(id+" "+username+" "+ Arrays.toString(hobby));
        return "form";

    }
```
## @RequestParam注解
建立请求参数和控制器方法形参的映射关系
三个属性:
	1. value : 指定形参赋值的参数的参数名
	2. required : 设置是否必须传此参数,默认值true
	3. defaultValue : 参数默认值

```Java
    @RequestMapping("test01test3")
    public String test04(@RequestParam(value = "id",required = true,defaultValue = "001")String id){
        System.out.println(id);
        return "form";
    }
```

##  @RequestHeader
建立请求头信息和控制器方法形参的映射关系

## @CookieValue
建立cookkie数据和控制器方法形参的映射关系

## 通过POJO获取请求参数
在控制器方法的形参位置设置一个实体类型的形参,若浏览器传输的请求参数的参数名和实体类中的属性名一致,那么请求参数就会为此属性赋值

解决获取请求参数乱码问题
* Get请求
直接设置tomcat目录下conf/server.xml文件，添加编码为utf-8
* Post请求
需要配置一个过滤器:
```xml
<!-- 配置编码过滤器 characterEncodingFilter 解决中文Post乱码问题  -->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <!-- 同时开启响应和请求的编码设置-->
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
```

# 域对象共享数据

## 使用ServletAPI
`request.setAttribute("lqw","097");`

## 使用ModelAndView(建议)
```Java
    @RequestMapping("data02")
    public ModelAndView data02(){
        ModelAndView mav=new ModelAndView();
        //向请求域共享数据
        mav.addObject("lqw","hello mav");
        //设置视图,即跳转页面
        mav.setViewName("index");
        return mav;
    }
```
## 使用model
```Java
    @RequestMapping("data03")
    public String data03(Model model){
        model.addAttribute("lqw","hello model");
        return "index";
    }
```

## 使用Map集合
```Java
    //使用Map集合
    @RequestMapping("data04")
    public String data04(Map<String,Object> mp){
        mp.put("lqw","hello map");
        return "index";
    }
```

## 使用ModelMap
```Java
    @RequestMapping("data05")
    public String data05(ModelMap modelMap){
        modelMap.addAttribute("lqw","hello modelmap");
        return "index";
    }
```

## Model/ModelMap/Map的关系
Model/ModelMap/Map 都是BindingAwareModelMap类
```Java
public interface Model{}
public class ModelMap extends LinkedHashMap<String,Object>{}
public class ExtendedModelMap extends ModelMap implements Model{}
public class BindingAwareModelMao extends ExtendedModelMap{}
```

## 向Session域共享数据
```Java
    @RequestMapping("data06")
    public String data06(HttpSession session){
        session.setAttribute("lqw","hello session");
        return "index";
    }
```
通过thymeleaf视图解析器在html页面中获取域数据
```html
<p th:text="${session.lqw}"></p>
```
## 向Application域共享数据
获取ServletContext对象设置域数据
```Java
    @RequestMapping("data07")
    public String data07(HttpSession session){
        session.getServletContext().setAttribute("lqw","hello Application");
        return "index";
    }
```
```html
<p th:text="${application.lqw}"></p>
```

# 视图
* SpringMVC中的视图是View接口，视图的作用渲染数据，将模型Model中的数据展示给用户
* SpringMVC视图的种类很多，默认有转发视图InternalResourceView和重定向视图RedirectView
* 当工程引入jstl的依赖，转发视图会自动转换为JstlView
* 若使用的视图技术为Thymeleaf,在SpringMVC配置文件中配置了Thymeleaf视图解析器,由此视图解析器解析后得到的是ThymeleafView

## ThymeleafView
当控制器方法中所设置的视图名称没有任何前缀时，此时的视图名称会被SpringMVC配置文件中所配置的视图解析器解析，视图名称拼接视图前缀和视图后缀所得到的最终路径，会通过转发的方式实现跳转

## 转发视图InternalResourceView
SpringMVC中默认的转发视图是InternalResourceView
SpringMVC中创建转发视图的情况:
当控制器方法中所设置的视图名称以"forward:"为前缀时，创建InternalResourceView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"forward:"去掉，剩余部分作为最终路径通过转发的方式实现跳转
```Java
    @RequestMapping("forwardview")
    public String forwardview(){
        return "forward:/view01";
    }
```

## 重定向视图RedirectView
SpringMVC中默认的重定向视图是RedirectView
当控制器方法中所设置的视图名称以"redirect."为前缀时，创建RedirectView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"redirect."去掉，剩余部分作为最终路径通过重定向的方式实现跳转
```Java
    @RequestMapping("redirctview")
    public String redirectview(){
        return "redirect:/view01";
    }
```

## 转发与重定向的区别
1. 从地址栏显示来说
forward是服务器请求资源,服务器直接访问目标地址的URL,把那个URL的响应内容读取过来,然后把这些内容再发给浏览器.浏览器根本不知道服务器发送的内容从哪里来的,所以它的地址栏还是原来的地址.
redirect是服务端根据逻辑,发送一个状态码,告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL.

2. 从数据共享来说
forward:转发页面和转发到的页面可以共享request里面的数据.
redirect:不能共享数据.

3. 从运用地方来说
forward:一般用于用户登陆的时候,根据角色转发到相应的模块.
redirect:一般用于用户注销登陆时返回主页面和跳转到其它的网站等

4. 从效率来说
forward:高.
redirect:低.

## 视图控制器view-controller
当控制器方法中,仅仅用来实现页面跳转,即只需要设置视图名称时,可以将控制器方法改为视图控制器
```xml
	<mvc:view-controller path="/viewcontroller" view-name="index"></mvc:view-controller>
```
在SpringMVC中设置任何一个view-controller时,其他控制器中的请求映射将全部失效,需要在配置文件中开启MVC注解驱动标签
```xml
<mvc:annotation-driven/>
```

# RESTFul
Representational State Transfer 表现层资源状态转移
a>资源
资源是一种看待服务器的方式，即，将服务器看作是由很多离散的资源组成。每个资源是服务器上一个可命名的抽象概念。因为资源是一个抽象的概念，所以它不仅仅能代表服务器文件系统中的一个文件、数据库中的一张表等等具体的东西，可以将资源设计的要多抽象有多抽象，只要想象力允许而且客户端应用开发者能够理解。与面向对象设计类似，资源是以名词为核心来组织的，首先关注的是名词。一个资源可以由一个或多个URI来标识。URI既是资源的名称，也是资源在Web上的地址。对某个资源感兴趣的客户端应用，可以通过资源的URI与其进行交互。

b>资源的表述
资源的表述是一段对于资源在某个特定时刻的状态的描述。可以在客户端-服务器端之间转移(交换)。资源的表述可以有多种格式，例如HTML/XMLIJSON/纯文本/图片/视频/音频等等。资源的表述格式可以通过协商机制来确定。请求-响应方向的表述通常使用不同的格式。

c>状态转移
状态转移说的是:在客户端和服务器端之间转移(transfer)代表资源状态的表述。通过转移和操作资源的表述，来间接实现操作资源的目的。

## RESTFul实现
四种基本操作:
    GET用来获取资源
    POST用来新建资源
    PUT用来更新资源
    DELETE用来删除资源
REST风格提倡URL地址使用统一的风格设计，从前到后各个单词使用斜杠分开，不使用问号键值对方式携带请求参数，而是将要发送给服务器的数据作为URL地址的一部分，以保证整体风格的一致性。

| 操作 | 传统方式 | REST风格 |
| ---- | -------- | -------- |
|查询全部|getAll|/user --> get请求方式|
|查询|getUserById?id=1| user/1 -->get请求方式|
|保存|saveUser|user --> post请求方式|
|删除|deketeUser?id=1|user/1 --> delete请求方式|
|更新|updateUser|user --> put请求方式|

## 实例
### 准备
#### bean
```Java
public class User {
    private int id;
    private String name;
    //基础方法省略
}
```
#### dao
```Java
@Repository
public class UserDao {
    private static Map<Integer, User> users=null;
    static {
        users=new HashMap<>();
        users.put(1001,new User(1001,"马云"));
        users.put(1002,new User(1002,"王健林"));
        users.put(1003,new User(1003,"王思聪"));
        users.put(1004,new User(1004,"马化腾"));
        users.put(1005,new User(1005,"比尔盖茨"));
        users.put(1006,new User(1006,"扎克伯格"));
        users.put(1007,new User(1007,"阿西吧"));
    }
    private static Integer initid=1008;
    public void save(User user){
        if(user.getId()==0){
            user.setId(initid++);
        }
        users.put(user.getId(),user);
    }

    public Collection<User> getAll(){
        return users.values();
    }

    public User get(Integer id){
        return users.get(id);
    }

    public void delete(Integer id){
        users.remove(id);
    }
}
```

### 查询全部
#### 静态页面
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>userlist</title>
</head>
<body>
    <table id="userstable" border="1" cellspacing="0" cellpadding="0" style="text-align: center">
        <tr>
            <th>user list</th>
        </tr>
        <tr>
            <th>id</th>
            <th>name</th>
        </tr>
        <tr th:each="user:${userlist}">
            <td th:text="${user.id}"></td>
            <td th:text="${user.name}"></td>
            <td>
                <a @click="deleteclick" th:href="@{/user/}+${user.id}">delete</a>
                <a th:href="@{/user/}+${user.id}">update</a>
            </td>
        </tr>
    </table>
    <form id="deleteform" method="post">
        <input type="hidden" name="_method" value="delete">
    </form>
    <script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
    <script type="text/javascript">
        var vue=new Vue({
            el:"#userstable",
            methods:{
                deleteclick:function(event){
                    //根据id获取表单元素
                    var deleteform=document.getElementById("deleteform");
                    //将触发点击事件的超链接赋值给表单的action事件
                    deleteform.action=event.target.href;
                    //提交表单
                    deleteform.submit();
                    //取消超链接默认行为
                    event.preventDefault();
                }
            }
        });

    </script>
</body>
</html>
```
#### 控制器方法
```Java
    @RequestMapping(value = "/user",method = RequestMethod.GET)
    public String getALl(Model model){
        model.addAttribute("userlist",userDao.getAll());
        return "userlist";
    }
```

### 删除
#### 控制器方法
```Java
    @RequestMapping(value = "/user/{id}",method = RequestMethod.DELETE)
    public String delete(@PathVariable("id")Integer id){
        userDao.delete(id);
        return "redirect:/user";
    }
```

### 添加
#### 静态页面
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>add user</title>
</head>
<body>
    <form method="post" th:action="@{/user}">
        id:<input type="text" name="id"><br/>
        name:<input type="text" name="name"><br/>
        <input type="submit" value="提交">
    </form>
</body>
</html>
```
#### 控制器方法
```Java
    @RequestMapping(value = "/user",method = RequestMethod.POST)
    public String add(User user){
        userDao.save(user);
        return "redirect:/user";
    }
```

### 回显和修改
#### 静态页面
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>update user</title>
</head>
<body>
<form method="post" th:action="@{/user}">
    <input type="hidden" name="_method" value="PUT">
    id:<input type="text" th:value="${user.id}" name="id"><br/>
    name:<input type="text" th:value="${user.name}" name="name"><br/>
    <input type="submit" value="修改">
</form>

</body>
</html>
```
#### 控制器方法
```Java
 @RequestMapping(value = "/user/{id}",method = RequestMethod.GET)
    public String getUserById(@PathVariable("id")Integer id,Model model){
        model.addAttribute("user",userDao.get(id));
        return "userupdate";
    }

    @RequestMapping(value = "/user",method = RequestMethod.PUT)
    public String update(User user){
        userDao.save(user);
        return "redirect:/user";
    }
```

# MVC注解驱动和静态资源放行
* 使用视图控制器需要开启MVC注解驱动,否则控制器方法会失效
* 使用静态资源(如删除操作中的vue.js)需要开放静态资源访问
```xml
	<!--mvc配置文件-->
    <!--开放对静态资源的访问-->
    <mvc:default-servlet-handler/>
    <!--开启MVC注解驱动-->
    <mvc:annotation-driven/>
```
## 静态资源放行

* 不使用 springmvc 框架,这些静态资源请求都会由 tomcat 的默认的 default 进行处理。
* 当配置的 DispatcherServlet 的映射路径不为 / 时，对静态资源的请求最终会由 tomcat 的默认配置来处理，所以不影响静态资源的正常访问。
* 如果配置的 DispatcherServlet 的映射路径为 / 时，会覆盖掉tomcat的默认的 default 配置，所以需要在 springmvc 文件中进行配置，对静态资源进行放行。

* 在 springmvc.xml 中放行 - 需要开启 <mvc:annotation-driven /> 注解驱动

### 1. 对全部资源放行
在springmvc文件中配置上 <mvc:default-servlet-handler/> ，发出静态资源请求后，请求传到 DispatcherServlet，DispatcherServlet 调用 RequestMappingHandlerMapping 进行映射匹配，匹配不成功，DispatcherServlet 最终会将请求转交给 tomcat 默认 default 进行处理。

### 2. 对指定目录下的资源放行 - <mvc:resources/>
如果配置了拦截器，需要在拦截器中进行过滤，否则会被拦截；
* 代表一级目录，** 代表多级目录
```xml
<mvc:resources location="/css/" mapping="/css/**" />
<mvc:resources location="/js/" mapping="/js/**" />
```

location元素表示webapp目录下的static包下的所有文件；
mapping元素表示以/static开头的所有请求路径，如/static/a 或者/static/a/b；
该配置的作用是：DispatcherServlet不会拦截以/static开头的所有请求路径，并当作静态资源
交由Servlet处理。

# HttpMessageConverter
HttpMessageConverter，报文信息转换器，将请求报文转换为Java对象，或将Java对象转换为响应报文HttpMessageConverter提供了两个注解和两个类型:@RequestBody，@ResponseBody，RequestEntity,ResponseEntity

## @RequestBody
@RequestBody可以获取请求体，需要在控制器方法设置一个形参，使用@RequestBody进行标识，当前请求的请求体就会为当前注解所标识的形参赋值
```Java
    @RequestMapping("/requestbody")
    public String requestBody(@RequestBody String requestBody){
        System.out.println(requestBody);
        return "form";
    }
```

## RequestEntity
RequestEntity封装请求报文的一种类型，需要在控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，可以通过getHeaders()获取请求头信息，通过getBody()获取请求体信息
```Java
    @RequestMapping("/requestentiyt")
    public String requestEntity(RequestEntity<String> request){
        System.out.println(request.getHeaders());
        System.out.println(request.getBody());
        System.out.println(request.getUrl());
        return "form";
    }
```

## @ResponseBody
@ResponseBody 表示该方法的返回结果直接写入 HTTP response body 中，一般在异步获取数据时使用(也就是AJAX)。
```Java
    @RequestMapping("/responsebody")
    @ResponseBody
    public String responseBody(){
        return "response body";
    }
```

### 处理json
1. 导入jackson-databind
2. 开启mvc注解驱动,此时HandlerAdaptor中会自动装配一个消息转换器:Mappingjackson2MassageConverter,可以将响应到浏览器的java对象转为json格式的字符串
3. 在处理器方法上使用@ResponseBody注解
4. 将Java对象直接作为控制器方法的返回值返回
```Java
    @RequestMapping("/responsebody01")
    @ResponseBody
    public Person responseBody01(){
        return new Person("01","person");
    }
```
浏览器中显示结果:{"id":"01","username":"person"}

### 处理ajax
```html
<div id="app">
        <a @click="testaxios" th:href="@{/testaxios}">test axios</a>
    </div>
    <script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
    <script type="text/javascript" th:src="@{/static/js/axios.min.js}"></script>
    <script type="text/javascript">
        new Vue({
            el:"#app",
            methods:{
                testaxios:function (event){
                    axios({
                        method:"post",
                        url:event.target.href,
                        params:{
                            id:"01",
                            name:"li"
                        }
                    }).then(function (response){
                        alert(response.data());
                    });
                    event.preventDefault();
                }
            }
        });
    </script>
```

```Java
    @RequestMapping("/testaxios")
    @ResponseBody
    public String testAxios(String id,String name){
        System.out.println(id+" "+name);
        return "success axios";
    }
```

## @RestController注解
@RestController注解是SpringMVC提供的一个复合注解,标识在控制器类上,相当于为类添加了@Controller注解,并且每个方法都添加了@ResponseBody注解.

## ResponseEntity
用于控制器方法的返回值类型,改造控制器方法的返回值就是响应到浏览器的响应报文

# 文件上传和下载
## 下载
```Java
@RequestMapping("/testDown")
    public ResponseEntity<byte[]> testDown(HttpSession session)throws IOException {
        //获取ServleyCOntext对象
        ServletContext servletContext=session.getServletContext();
        //获取服务器中文件的真实路径
        String readPath=servletContext.getRealPath("/static/image/美女.jpg");
        //创建输入流
        InputStream is=new FileInputStream(readPath);
        //创建字节数组
        //available() 获取输入流长度
        byte[] bytes=new byte[is.available()];
        //将字节流读到字节数组中
        is.read(bytes);
        //创建HttpHeaders对象设置响应头信息
        MultiValueMap<String,String> headers=new HttpHeaders();
        //设置下载方式和下载文件名
        headers.add("Content-Disposition","attachment;filename=美女.jpg");
        //设置响应头状态码
        HttpStatus status=HttpStatus.OK;
        //创建ResponseEntity对象
        ResponseEntity<byte[]> response=new ResponseEntity<>(bytes,headers,status);
        is.close();
        return response;
    }
```

## 上传
1. 依赖 commons-fileupload
2. 配置mvc文件上传解析器
```xml
<!--一定要配置id="multipartResolver" spring根据id获取该解析器-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"> </bean>
```
3. 前端页面
```html
    <div>
        <form th:action="@{/testUpload}" method="post" enctype="multipart/form-data">
            <input type="file" name="photo"><br/>
            <input type="submit" value="提交">
        </form>
    </div>
```
4. 控制器方法
```Java
    @RequestMapping("/testUpload")
    public String testUpload(@RequestParam(value = "photo")MultipartFile photo, HttpSession session)throws IOException{
        String filename=photo.getOriginalFilename();
        //获取文件后缀名
        String suffixname=filename.substring(filename.lastIndexOf("."));
        //System.out.println(filename);
        String uuid= UUID.randomUUID().toString();
        filename=uuid+suffixname;
        String realPath=session.getServletContext().getRealPath("photo");
        //System.out.println(realPath);
        File file=new File(realPath);
        if(!file.exists()){
            file.mkdir();
        }
        String path=realPath+File.separator+filename;
        //System.out.println(path);
        photo.transferTo(new File(path));
        return "index";
    }
```

# 拦截器
## 配置拦截器
* SpringMVC中的拦截器用于拦截控制器方法的执行
* SpringMVC中的拦截器需要实现HandlerInterceptor或者继承HandlerInterceptorAdapter类
* SpringMVC的拦截器必须在SpringMVC的配置文件中进行配置:
```xml
    <!--配置拦截器-->
    <mvc:interceptors>
        <!--方法一: 拦截所有请求-->
        <bean class="com.lqw.mvc.interceptor.FirstInterceptor"> </bean>
    </mvc:interceptors>
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/*"/>
            <!--取消拦截目录-->
            <mvc:exclude-mapping path="/"/>
            <ref bean="firstInterceptor"></ref>
        </mvc:interceptor>
    </mvc:interceptors>
```
## 拦截器三个抽象方法
SpringMVC中的拦截器有三个抽象方法:

1. preHandle:控制器方法执行之前执行preHangdle()，其boolean类型的返回值表示是否拦截或放行，返回true为放行，即调用控制器方法;返回false表示拦截，即不调用控制器方法
2. postHandle:控制器方法执行之后执行postHandle()
3. afterComplation:处理完视图和模型数据，渲染视图完毕之后执行afterComplation()

## 多个拦截器执行顺序
1. 若每个拦截器的preHandle()都返回true
此时多个拦截器的执行顺序和拦截器在SpringMVC的配置文件的配置顺序有关:
    * preHandle()会按照配置的顺序执行
    * 而postHandle()和afterComplation()会按照配置的反序执行

2. 若某个拦截器的preHandle()返回了false
    * preHandle()返回false和它之前的拦截器的preHandle()都会执行
    * postHandle()都不执行
    * 返回false的拦截器之前的拦截器的afterComplation()会执行

# 异常处理器
## 基于配置的异常处理
SpringMVC提供了一个处理控制器方法执行过程中所出现的异常的接口:HandlerExceptionResolver

HandlerExceptionResolver接口的实现类有:DefaultHandlerExceptionResolver和
SimpleMappingExceptionResolver

SpringMVC提供了自定义的异常处理器SimpleMappingExceptionResolver，使用方式:
```xml
    <!--配置异常处理-->
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="exceptionMappings">
            <props>
                <!--key = 异常全类名
                标签内容为异常跳转视图名称
                -->
                <prop key="java.lang.ArithmeticException">error</prop>
            </props>
        </property>
        <!--在请求域中添加键,键为value,用于显示异常信息-->
        <property name="exceptionAttribute" value="exception"> </property>
    </bean>
```

## 基于注解的异常处理
```Java
@ControllerAdvice
public class ExceptionController {
    @ExceptionHandler(value = {ArithmeticException.class})
    public String testException(Exception e, Model model){
        model.addAttribute("exception",e);
        return "error";
    }
}

```

# 注解配置(完全注解开发)
使用配置类和注解代替SpringMVC配置文件和web.xml的功能

## 1. 创建初始化类 代替web.xml
在Servlet3.0环境中，容器会在类路径中查找实现javax.servlet.ServletContainerInitializer接口的类，如果找到的话就用它来配置Servlet容器。
Spring提供了这个接口的实现，名为SpringServletContainerInitializer，这个类反过来又会查找实现WebApplicationInitializer的类并将配置的任务交给它们来完成。Spring3.2引入了一个便利的
WebApplicationInitializer基础实现，名为AbstractAnnotationConfigDispatcherServletInitializer，当我们的类扩展了AbstractAnnotationConfigDispatcherServletlnitializer并将其部署到Servlet3.0容器的时候，容器会自动发现它，并用它来配置Servlet上下文。
```Java
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer{
    /**
     * 指定spring配置类
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    /**
     * 指定SpringMVC配置类
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    /**
     * 指定DispatcherServlet的映射规则 即url-pattern
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    /**
     * 注册过滤器
     * @return
     */
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter characterEncodingFilter=new CharacterEncodingFilter();
        characterEncodingFilter.setEncoding("UTF-8");
        characterEncodingFilter.setForceResponseEncoding(true);
        HiddenHttpMethodFilter hiddenHttpMethodFilter=new HiddenHttpMethodFilter();
        return new Filter[]{characterEncodingFilter,hiddenHttpMethodFilter};
    }
}
```


## 2.创建Web.config类 代替SpringMVC配置文件

```Java
/**
 * 代替SpringMVC配置文件
 * 1.扫描组件 2.视图解析器 3.view-controller 4.default-servlet-handler
 * 5.mvc注解驱动 6.文件上传解析器 7.异常处理 8.拦截器
 */
//标记当前为配置类
@Configuration
//1.扫描组件
@ComponentScan("com.lqw.mvc")
//5. mvc注解驱动
public class WebConfig implements WebMvcConfigurer {
    //2. 视图解析器
    //配置生成模板解析器
    @Bean
    public ITemplateResolver templateResolver(){
        WebApplicationContext webApplicationContext= ContextLoader.getCurrentWebApplicationContext();
        ServletContextTemplateResolver templateResolver=new ServletContextTemplateResolver(webApplicationContext.getServletContext());
        templateResolver.setPrefix("WEB-INF/templates");
        templateResolver.setSuffix(".html");
        templateResolver.setCharacterEncoding("UTF-8");
        templateResolver.setTemplateMode(TemplateMode.HTML);
        return templateResolver;
    }

    //生成模板引擎并为模板引擎注入模板解析器
    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver){
        SpringTemplateEngine templateEngine=new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        return templateEngine;
    }

    //生成视图解析器并为解析器注入模板引擎
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine){
        ThymeleafViewResolver viewResolver=new ThymeleafViewResolver();
        viewResolver.setCharacterEncoding("UTF-8");
        viewResolver.setTemplateEngine(templateEngine);
        return viewResolver;
    }

    //4. default-servlet-handler 静态资源放行
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    // 8.拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        MyInterceptor myInterceptor=new MyInterceptor();
        registry.addInterceptor(myInterceptor).addPathPatterns("/");
    }

    // 3. view-controller 视图控制器
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/hello").setViewName("hello");
    }

    //6.文件上传解析器
    @Bean
    public MultipartResolver multipartResolver(){
        CommonsMultipartResolver commonsMultipartResolver=new CommonsMultipartResolver();
        return commonsMultipartResolver;
    }

    //7.异常处理

    @Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        WebMvcConfigurer.super.configureHandlerExceptionResolvers(resolvers);
        SimpleMappingExceptionResolver exceptionResolver=new SimpleMappingExceptionResolver();
        Properties properties=new Properties();
        properties.setProperty("java.lang.ArithmeticException","error");
        exceptionResolver.setExceptionMappings(properties);
        exceptionResolver.setExceptionAttribute("exception");
        resolvers.add(exceptionResolver);
    }
}
```

## 配置文件改写配置类方法

* \<bean>标签一般可以通过@Bean注解的方法改写,方法放回值为bean对象

* 非bean对象标签,一般通过覆写配置类实现的WebMvcConfigure接口的抽象方法实现.

|XML	|java类	|备注|
|----|----|----|
|beans标签	|@Configuration	|通常一个XML文件对应一个@Configuration|
|bean标签	|@Bean或者@Component(“id名”)|	方法用@Bean，类用@Component|
|id	|方法名| |	
|class	|方法的返回值类型	| |
|property 标签	|类定义的属性	| |
|constructor-arg 标签	|类构造器的参数| |	


# SpringMVC执行流程

## SpringMVC常用组件
* DispatcherServlet:前端控制器，不需要工程师开发，由框架提供
作用:统一处理请求和响应，整个流程控制的中心，由它调用其它组件处理用户的请求

* HandlerMapping:处理器映射器，不需要工程师开发，由框架提供
作用:根据请求的url、method等信息查找Handler，即控制器方法. 

* Handler:处理器，需要工程师开发
作用:在DispatcherServlet的控制下Handler对具体的用户请求进行处理. HandlerAdapter:处理器适配器，不需要工程师开发，由框架提供作用:通过HandlerAdapter对处理器(控制器方法)进行执行

* ViewResolver:视图解析器，不需要工程师开发，由框架提供
作用:进行视图解析，得到相应的视图，例如: ThymeleafView、InternalResourceView、RedirectView. 

* View:视图，不需要工程师开发，由框架或视图技术提供
作用:将模型数据通过页面展示给用户

## 执行流程
1) 用户向服务器发送请求，请求被SpringMVC 前端控制器DispatcherServlet捕获。
2) DispatcherServlet对请求URL进行解析，得到请求资源标识符(URI)，判断请求URI对应的映射:
    a)不存在
        i.再判断是否配置了mvc:default-servlet-handler
        ii.如果没配置，则控制台报映射查找不到，客户端展示404错误
        iii.如果有配置，则访问目标资源(一般为静态资源，如:JS.CSS,HTML)，找不到客户端也会展示404错误
    b)存在:则执行下面的流程

3) 根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象I(包括Handler对象以及Handler对象对应的拦截器)，最后以HandlerExecutionChain执行链对象的形式返回。

4) DispatcherServlet根据获得的Handler，选择一个合适的HandlerAdapter。

5) 如果成功获得HandlerAdapter，此时将开始执行拦截器的preHandler(...)方法【正向】

6) 提取Request中的模型数据，填充Handler入参，开始执行Handler (Controller)方法，处理请求。在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作:
    a) HttpMessageConveter:将请求消息(如]son、xml等数据）转换成一个对象，将对象转换为指定的响应信息
    b)数据转换:对请求消息进行数据转换。如String转换成Integer、Double等
    c)数据格式化:对请求消息进行数据格式化。如将字符串转换成格式化数字或格式化日期等
    d)数据验证:验证数据的有效性（长度、格式等)，验证结果存储到BindingResult或Error中

7) Handler执行完成后，向DispatcherServlet返回一个ModelAndView对象。

8) 此时将开始执行拦截器的postHandle(...)方法【逆向】。

9) 根据返回的ModelAndView (此时会判断是否存在异常:如果存在异常，则执行HandlerExceptionResolver进行异常处理)选择一个适合的ViewResolver进行视图解析，根据Model和View，来渲染视图。

10) 渲染视图完毕执行拦截器的afterCompletion(...)方法【逆向】。

11) 将渲染结果返回给客户端。









