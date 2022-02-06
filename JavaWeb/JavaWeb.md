# Javaweb学习笔记

# Javascript

## 变量类型：
   * 数值：number
   * 字符串：string
   * 对象：object
   * 布尔：boolean
   * 函数；function
## 特殊值：
   * undefine
   * null
   * NAN 非数值类型
## 运算符：
   * == 等于：数值相等
   * === 全等于，数值、数据类型都相等
## 逻辑运算：
   所有类型都可以作为boolean变量使用，null、undefined、0、“”空串都为false；
   &&：
       * 当表达式全为真返回最后一个表达式的值
       * 当表达式中存在假，返回第一个为假的表达式的值
   ||：
       * 当表达式全为假返回最后一个表达式的值
       * 当表达式中存在真，返回第一个为真的表达式的值
## 数组：
   `var 数组名=[];`//定义一个空数组

   ` var 数组名=new Array();`

   `数组名[3] =“string”` //可以通过任意下标赋值，数组自动扩容，未赋值内为undefined

## 函数：
   `function 函数名(a,b){}` //定义函数方式一,形参不需要声明类型
   `var 函数名 =function(形参列表){}` //定义函数方式二
   JS不允许函数重载
   隐藏参数：argument
## 自定义对象：
   方法一：
   `var 变量名=new Object();`  //实例对象
   `变量名.属性名=值;`           //定义一个属性
   `变量名.函数名=function(){}`  //定义一个函数
   方法二：
```javascript
    var 变量名={
       属性名:值,
       函数名:function(){}
   }
```
## 事件：输入设备与页面进行交互的反应
注册事件（绑定）：事件响应后的操作代码
		静态注册事件：通过html标签的事件属性直接赋予事件响应后的代码
		动态注册事件：先通过js代码得到标签的dom对象，再通过`dom对象.事件名			=function(){}`赋予事件响应后的代码
        1、获取标签对象
        2、标签对象.事件名=function(){}    
        常用事件：
            onload：加载完成事件
            onclick：单击事件
            onblur；失去焦点事件
            onchange：内容发生改变事件
            onsubmit：表单提交事件
        DOM模型：
            Document对象：
            document.getElementById();//返回第一个结果
            document.getElementByName();//返回结果集合
            document.getElementByTagName();///返回结果集合
            优先级顺序：id>name>tagName
            一定要在页面加载完成才能查  
```javascript
//静态注册onload事件
document.write("<h1>大标题</h1>")
function onloadFunc(){
    alert("静态页面加载完毕")
}
//动态注册onload事件
window.onload=function (){
    alert("动态页面加载完毕");
    //动态注册onclick事件
    var buttonobj=document.getElementById("button1");//获取标签对象
    buttonobj.onclick=function (){
        alert("动态注册onclick事件");
    }
    //动态注册失去焦点事件
    var textobj=document.getElementById("text1");
    textobj.onblur=function (){
        var  d = document.getElementById("span1");
        d.innerHTML="请输入";
    }
}
```

# jQuery

## jQuery对象和DOM对象
   jQuery对象=DOM对象数组+jQuery方法 
   jQuery对象和DOM对象属性、方法不能互用
   >转换
   >>jQuery--> `DOM jQuery对象[下标]`
   >>DOM--》jQuery `$(DOM对象)`

## 核心函数：$()  
 1. 传入参数为函数`$(函数)`，表示页面加载完成之后，相当于`window.onload=function(){}`
 2. 传入参数为**HTML字符串**，根据字符串创建节点元素
 3. 传入参数为**选择器字符串** 返回查询结果  
    $("#id"); $("标签名"); $(".class");
 4.   传入参数为**DOM对象** 会把DOM对象转换为jQuery对象  
## 选择器
### 基本选择器
 1. 元素选择器
    $("p") 选取 <p> 元素。
    $("p.intro") 选取所有 class="intro" 的 <p> 元素。
    $("p#demo") 选取所有 id="demo" 的 <p> 元素。
 3. CSS选择器
    ` $("p").css("background-color","red"); `  

### 过滤选择器
1. 基本过滤选择器
2. 内容过滤选择器  
   `$("div:contains('内容')");`
   `$("div:empty");`//空
   `$("div:parent");`//非空
   `$("div:has(div)");//返回含有选择器所匹配的元素的元素`
3. 属性过滤选择器
   通过正则表达式过滤
   `$("div[href]")` 选取所有带有 href 属性的元素。
   `$("[href='#']")` 选取所有带有 href 值等于 "#" 的元素。
   `$("[href!='#']")` 选取所有带有 href 值不等于 "#" 的元素。
   `$("[href$='.jpg']")` 选取所有 href 值以 ".jpg" 结尾的元素。
4. 表单过滤选择器  
   `$(":type属性")`

## 元素的筛选
`$("选择器").筛选函数`;

## 属性操作
不传参是获取，传参是设置：
* `html()` 获取和获取头尾标签之间内容，和DOM innerHTML相同
* `text()` 设置和获取头尾标签之间的文本，和DOM innerTest相同
* `val()` 设置和获取表单项的value属性值，和DOM value相同
* `attr("属性名"[,"属性值"])`  操作属性值，不推荐操作checked、readOnly、selected、disable等，可操作自定义属性值
* `prop("属性名"[,"属性值"])`   与attr互补，该属性值为true|false  
## DOM的CRUD
### 增
内部插入
* `a.appendTo(b)` 把 a元素插到b元素末尾，成为最后一个子元素
* `a.prependTo(b)`把 a元素插到b元素前面，成为第一个子元素
外部插入
* `a.insertBefore(b)`
* `a.insertAfter(b)`
### 替换
* `a.replaceWith(b)` 用b替换a
* `a.replaceAll(b)`  用a替换所有b
### 删除
* `a.remove()`  删除a标签
* `a.empty()` 清空a标签里的内容

## CSS样式操作
* `a.addClass("")` 添加样式
* `a.removeClass("")`删除样式
* `a.toggleClass("")`添加切换样式
* `offset()` 获取和设置元素坐标

## 动画操作
* `show()`
* `hide()`
* `toggle()`
* `fadeIn()` 淡入
* `fadeOut()` 淡出
* `fadeTo()` 在指定时长慢慢将透明的调整到指定的值
* `fadeToggle()` 切换
  以上函数可以选择传入两个参数
    1、speed（毫秒） 动画执行时间
    2、callback 回调函数（动画完成后执行的函数）

## jQuery事件操作
### jQuery与原生JS加载的区别
>什么时候触发？
>>jQuery在浏览器的内核解析完页面的标签创建好DOM对象后立即执行
>>原生JS在浏览器的内核解析完页面的标签创建好DOM对象、等标签显示需要的内容加载后执行  

>触发顺序
>
>>jQuery先于原生JS执行  

>执行次数
>>jQuery会把全部注册的function函数一次执行
>>原生JS只会执行最后一次赋值函数

### 常用事件
1. click()  绑定并触发单击事件
2. mouseover() 鼠标移入事件
3. mouseout() 鼠标移出事件
4. bind("event1 event2...",function()) 一次绑定多个事件
5. one() 与bind 用法相同，只执行一次绑定事件
6. unbind() 解绑
7. live() 动态绑定事件

### 事件冒泡
>概念：
>
>>父子元素同时监听一个事件，当触发子元素事件时，父元素也响应同一元素  

>如何阻止事件冒泡：
>
>>在子元素事件函数内添加：return false  

### Javascript事件对象
>概念：
>
>>封装有触发事件信息的JavaScript对象  

>如何获取？
>
>>在给元素绑定事件的时候，在事件的function(event) 参数列表中添加一个参数event，event就是传递事件处理函数的事件对象

# XML

## XML简介
>什么是XML
>
>>XML是可拓展的标记性语言

>XML的作用
>>1. 保存数据,且这些数据具有自我描述性
>>2. 作为项目或模块的配置文件
>>3. 作为网络传输数据的模式(现在以JSON为主)

## 语法
1. 声明 `<?xml version="1.0" encoding="utf-8" ?>`
2. 注释 `<!---->`
3. 标签(元素) 与HTML大致一致 
4. 文本区域 `<![ADATA[纯文本]]> `不需要转义的纯文本格式

## XML解析

* W3C制定的DOM
* SUM公司制定的SAX (Simple API for XML)
* 第三方解析 
    jdom在dom基础上进行封装
    dom4j在jdon基础上进行封装
    pull主要用于Android开发,与SAX相似

## dom4j解析技术
```java

SAXReader saxreader = new SAXReader();
Document document=saxreader.read("java_test/src/book.xml");
//通过Element对象获取根元素
Element root=document.getRootElement();
//通过根元素获取book标签对象 注意区分element()与elements()方法
List<Element> books=root.elements("book");
for(Element book:books){
Element nameElement=book.element("name");
//asXML()方法,返回标签对象的字符串
//nameElement.asXML();
//获取标签的Element对象的内容的文本
String name=nameElement.getText();
//直接获取指定标签名的文本内容
String price =book.elementText("price");
//获取属性值
String sn=book.attributeValue("sn");
System.out.println(new Book(sn,name,price));

```

``` xml
<?xml version="1.0" encoding="utf-8" ?>
<books>
    <book sn="0004">
        <name>时间简史</name>
        <author>史蒂芬霍金</author>
        <price>$80.0</price>
    </book>
</books>
```

# TomCat
> 由Apache组织提供的一种web服务器,提供对jsp和Servltet支持,是一种轻量级的javaweb容器(服务器).

## 修改Tomcat端口号
找到conf/server.xml , 修改Connector port="端口号"

## 部署web到Tomcat
方法一:  把web工程目录拷贝到Tomcat的webapps目录下
      通过http://ip:端口号/工程路径 访问
方法二: 找到 \conf\Catalina\localhost , 创建.xml配置文件:
```xml
<Contest path="/配置文件名" docBase="配置文件目录" />
```

# Servlet
Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

## 实现Servlet程序

### 方法一:实现Servlet接口
1. 编写类实现Servlet接口
2. 重写service方法,处理请求,响应数据
3. 在web.xml 中配置Servlet程序访问的地址

```java
public class ServletDemo1 implements Servlet {
    //public ServletDemo1(){}
     //生命周期方法:当Servlet第一次被创建对象时执行该方法,该方法在整个生命周期中只执行一
    public void init(ServletConfig arg0) throws ServletException {}
    //生命周期方法:对客户端响应的方法,该方法会被执行多次，每次请求该servlet都会执行该方法
    public void service(ServletRequest arg0, ServletResponse arg1)
            throws ServletException, IOException {}
    //生命周期方法:当Servlet被销毁时执行该方法
    public void destroy() {}
//当停止tomcat时也就销毁的servlet。
    public ServletConfig getServletConfig() {
        return null;
    }
    public String getServletInfo() {
        return null;
    }
}
```
```xml
    <!--servlet标签给Servlet标签给Tomcat配置Servlet程序-->
    <servlet>
        <!--servlet-name 标签 给Servlet程序起一个别名 一般是类名-->
        <servlet-name>HelloServlet</servlet-name>
        <!--servlet-class 是Servlet程序的全类名-->
        <servlet-class>HelloServlet</servlet-class>
    </servlet>
    <!-- servlet-mapping标签给Servlet程序配置访问地址-->
    <servlet-mapping>
        <!--告诉服务器 当前配置的地址给哪个Servlet程序使用-->
        <servlet-name>HelloServlet</servlet-name>
        <!-- 配置访问地址
            /  表示 http://ip:port/工程路径
            /hello 表示 http://ip:port/工程路径/hello
        -->
        <url-pattern>/hello/</url-pattern>
    </servlet-mapping>
</web-app>
```

### 方法二:继承HttpServlet
1. 编写类继承HttpServlet类
2. 根据业务需求重写doGet()和doPost()方法
3. 在web.xml 中配置Servlet程序访问的地址

### web.xml配置文件
web.xml文件是用来初始化配置信息：比如Welcome页面、servlet、servlet-mapping、filter、listener、启动加载级别等。

#### 常用web.xml标签功能
1. 指定欢迎页面
2. 命名与定制URL
3. 定制初始化参数
4. 指定错误处理页面
5. 设置过滤器
6. 设置监听器
7. 设置会话(Session)过期时间

## Servlet程序继承关系
<Interface> Servlet   <----    <Class> GenericServlet    <----    <Class> HttpServlet
<Interface> Servlet: Servlet接口，只是负责定义Servlet程序的访问规范。
<Class> GenericServlet: 做了很多空实现。并持有一个ServletConfig类的引用。并对ServletConfig的使用做一些方法。
<Class> HttpServlet:  实现了请求的分发处理,HttpServlet抽取类实现了服务(方法，并实现了请求的分发处理)

## ServletConfig类
Servlet程序的配置信息类

### 作用
1. 获取Servlet程序的别名
2. 获取初始化参数init-param
3. 获取ServletContext对象

## ServletContext接口
1. 表示Servlet上下文对象
2. 一个web工程,只有一个ServletContext对象
3. ServletContext对象是域对象

ServletContext作用
1. 获取web.xml中配置的上下文参数context-param
2. 获取当前的工程路径，格式:/工程路径
3. 获取工程部署后在服务器硬盘上的绝对路径
4. 像Map一样存取数据


## HTTP协议
### 请求
除了form中method="POST",其余一一般都为GET请求
#### GET请求
1. 请求行
    (1)请求的方式  GET
    (2)请求的资源路径[+?+请求参数]
    (3)请求的协议版本号
    `GET /untitled_war_exploded/hello HTTP/1.1`
2. 请求头
    key:value

#### POST请求
1.请求行
2.请求头
3.空行
3.请求体

### 响应
#### 响应HTTP协议格式
1.响应行
    (1)响应的协议和版本号
    (2)响应状态码
        常见响应状态码
            200:请求成功
            302:请求重定向
            404:服务器收到请求,请求数据不存在(请求地址错误)
            500:服务器内部错误
    (3)响应状态描述符
2.响应头
    key:value
3.空行
4.响应体:回传给客户的数据

### HttpServletRequest类
Tomcat服务器把请求过来的Http协议信息解析好封装到Request对象中,然后传递到service(doGet和doPost)方法中,可以通过HttpServletRequest对象获取所有请求信息

#### 常用方法
i.   getRequestURI() 获取请求的资源路径
ii.   getRequestURL() 获取请求的统—资源定位符（绝对路径)
iii.  getRemoteHost() 获取客户端的IP地址
iv.   getHeader() 获取请求头
v.   getParameter() 获取请求的参数
vi.  getParameterValues() 获取请求的参数(多个值的时候使用)
vii.  getMethod() 获取请求的方式GET 或POST
viii.   setAttribute(key , value); 设置域数据
ix.   getAttribute(key ); 获取域数据
x.   getRequestDispatcher() 获取请求转发对象
xi.  setCharacterEncoding("UTF-8");  解决POST请求中中文乱码问题

### HttpServletResponse类
HttpServletResponse 接口继承自 ServletResponse 接口，主要用于封装 HTTP 响应消息。

####两个输出流
两个流不能同时使用
1. 字节流  getOutputStream();   常用于下载(传递二进制数据)
2. 字符流  getWriter(0; 常用于回传字符串(常用)

### MINE类型说明
MINE是http协议中数据类型,格式:"大类型/小类型"

### 请求转发
服务器收到请求后从一个资源跳转到另一个资源的操作
```java
RequestDispatcher requestDispatcher=request.getRequestDispatcher("/index.jsp");
requestDispatcher.forward(request,response);
```

base标签
base标签设置页面相对路径工作时的参照地址,href属性就是参照的地址值  

### / 斜杠的不同含义
1. 在浏览器中解析:http://ip:port/
2. 在服务器中解析:http://ip:port/工程路径/

### 解决响应中文乱码问题
方法一:
```java
        //设置服务器字符集为UTF-8
        response.setCharacterEncoding("UTF-8");
        //通过响应头设置浏览器使用UTF-8字符集
        response.setHeader("Content-Type","text/html;charset=UTF-8");
```
方法二:
此方法一定要在获取流对象前使用
```java
        //同时设置服务器 客户端 响应头为UTF-8字符集
        response.setContentType("text/html;charset=UTF-8");
```

### 请求重定向
服务器收到客户端请求,给客户端一个新的访问地址;
    服务器发给客户端:响应状态码302;Location响应头:新地址
    可以访问工程外资源
方法一:
```java
	//设置相应状态码
	response.setStatus(302);
	//设置响应头
	response.setHeader("Location","新地址");
```
方法二(推荐):
```java
	response.setRedirect("新地址");
```

# JSP
Java Server Pages，是一种动态网页开发技术。它使用JSP标签在HTML网页中插入Java代码。
JSP是一种Java servlet，主要用于实现Java web应用程序的用户界面部分。网页开发者们通过结合
JSP通过网页表单获取用户输入数据、访问数据库及其他数据源，然后动态地创建网页。

## page指令
`<%@ page contentType="text/html;charset=UTF-8" language="java" %>`
### 常见属性
1. language 表示jsp翻译后是什么语言文件,暂时只支持java
2. contenttype 表示jsp返回的数据类型
3. pageEncoding 表示jsp页面本身的字符集
4. import 等于java中的import
5. aotuFlush 表示是否自动刷新缓冲区 默认值是true
6. buffer 设置缓冲区大小,默认是8kb
7. errorPage 设置jsp页面运行出错时,自动跳转的页面路径
8. isErrorPage 设置当前页面是否是错误信息页面
9. session 设置当前访问页面是否会自动创建HttpSession对象,默认是true
10. extends 设置jsp翻译出来的java类默认继承对象

## 常用脚本
### 声明脚本
`<%!  声明java代码  %>`
`<%!  类属性  %>`
`<%!  static静态代码块  %>`
`<%!  类方法  %>`
`<%!  内部类  %>`
### 表达式脚本
`<%=       表达式          >`
作用:在jsp页面上输出 数据
* 所有的表达式脚本都会被翻译到_jspService方法中
* 表达式脚本都会被翻译成为out.print()输出到页面上
* 表达式脚本不能以分号为结尾
### 代码脚本
`<%    java语句    %>`
* 代码脚本都会被翻译到_jspService方法中
* 代码脚本可以由多个代码脚本块/表达式脚本完成一个java语句

## jsp中三种注释
html注释会被翻译到java源代码中,以out.write()输出到客户端
java注释会被翻译到java源代码中
jsp注释 <%--jsp注释 --%>

## 9个内置对象
request 请求对象
response 响应对象
pageContext JSP的上下文对象
session 会话对象
application ServletContext对象
config ServletConfig对象
out jsp输出流对象
page 指向当前jsp的对象
exception 异常对象

### 四大域对象:
功能相同,存取数据范围不一致 ,使用优先级从小到大             
pageContext(PageContextlmpl类)         当前jsp有效
request(HttpServletRequest类)              一次请求有效
session(HttpSession类)                            一次会话(打开浏览器访问服务器到关闭浏览器)有效
application(ServietContext类)               整个web工程有效

### out和response
当jsp页面中所有代码执行完成之后，会做以下两个动作：
1、执行out.flush()操作，会把out缓冲区中的数据追加到response缓冲区末尾
2、执行response的刷新操作，把全部数据写给客户端
由于jsp翻译之后，底层源代码都是使用out来进行输出，所以一般情况下，在jsp页面中统一使用out来进行输出。避免打乱页面输出内容的顺序。
而对于out输出的方法，又有两种：
out.write()：该方法对字符串输出有效
out.print()：该方法对任意数据类型输出都有效
源码中，print是将其他几种数据类型都转换成字符串类型，然后调用write()输出的；而write()中，对于整型的输出，他是先转换成字符型，放入其缓冲数组中，然后再输出，此时输出的就是该数值对应的ASCII码的字符。

## jsp常用标签
### 静态包含
`<%@ include="/path"%>`
* 静态包含不会翻译被包含的jsp页面
* 静态包含本质是把被包含的jsp页面代码拷贝到包含的位置执行输出

### 动态包含
`<jsp:include page="/path"></jsp:include>`
* 动态包含会把jsp页面翻译成java代码
* 动态包含底层是调用java代码执行输出jsp页面
* 动态包含可以带参数

### 请求转发
<jsp:forward page="/path"><jsp:forward>

# EL表达式
Expression Language 表达式语言
替代jsp页面中的表达式脚本进行数据输出,主要用于输出域对象中的数据.
${}
## 运算
1. empty运算
    可以判断一个数据是否为空
    以下几种情况为空:
    1、值为null 值的时候，为空
    2、值为空串的时候，为空
    3、值是 object类型数组，长度为零的时候
    4、list集合，元素个数为零
    5、map集合，元素个数为零
2.  三元/关系/逻辑/算术 运算 与java中一致
3. []中括号运算:
    * 输出有序集合中对应下标的值
    * 输出map集合中key里含有特殊字符的key的值
## 11个隐含对象
| 变量 | 类型 | 作用 |
| ---- | ---- | ---- |
|pageContext|PageContextlmpl|它可以获取jsp中的九大内置对象|
|pagescope|Map<String,object>|它可以获取pageContext 域中的数据|
|requestScope|Map<String,object>|它可以获取 Request 域中的数据|
|sessionscope|Map<String.Object>|它可以获取session域中的数据|
|applicationScope|Map<String,Object>|它可以获取ServletContext 域中的数据|
|param|Map<String,String>|它可以获取请求参数的值|
|paramValues|Ma<String,String[]>|它也可以获取请求参数的值，获取多个值的时候使用|
|header|Map<String,String>|它可以获取请求头的信息|
|headervalues|Map<String,String[]>|它可以获取请求头的信息，它可以获取多个值的情况|
|cookie|Map<String,Cookie>|它可以获取当前请求的cookie 信息|
|initParam|Map<String,Cookie>|它可以获取当前请求的Cookie信息|

# JSTL标签库
代替代码脚本
依赖:taglibs-standard-impl.jar       taglibs-standard-spec.jar
## set标签
往域中保存数据
`<c:set scope="page" var="lqw01" value="09701" /> `
scope : 域
var : key
value : value

## if标签
做if判断
`<c:if test="EL表达式"/>`

## choose/when/otherwise标签
```JSTL
    <c:choose>
        <c:when test="${条件1}">动作1</c:when>
        <c:when test="${条件2}">动作2</c:when>
        <c:otherwise>其余动作</c:otherwise>
    </c:choose>
```

## foreach标签
```JSTL
    <c:forEach begin="1" end="10" var="i">
        <h1>${i}</h1>
    </c:forEach>
```
item属性:遍历对象类型数组时的数据源
var : 遍历到的数据
begin : 开始索引值
end : 结束索引值
step : 遍历步长值
varStatus : 当前遍历到的数据的状态

# 文件的上传和下载
## 上传
1、要有一个form标签，method=post请求
2、form标签的encType属性值必须为multipart/form-data值
3、在form标签中使用input type=file添加上传的文件
4、编写服务器代码(servlet程序）接收，处理上传的数据。
encType=multipart/form-data表示提交的数据，以多段(每一个表单项一个数据段〉的形式进行拼接，然后以二进制流的形式发送给服务器

## 解析上传数据
依赖:commons-fileupload.jar    commons-io.jar
```java
        //先判断数据是否是多段的,只有多段数据才是文件上传的
        if(ServletFileUpload.isMultipartContent(req)){
            //创建FileItemFactory工厂实现类
            FileItemFactory fileItemFactory=new DiskFileItemFactory();
            //创建用于解析上传数据的工具类
            ServletFileUpload servletFileUpload=new ServletFileUpload(fileItemFactory);
            //解析上传数据,得到每一个表单项
                List<FileItem> list=servletFileUpload.parseRequest(req);
                //判断是普通表单项还是文件
                for(FileItem it:list){
                    if(it.isFormField()){
                        System.out.println(it.getFieldName());
                        System.out.println(it.getString("UTF-8"));
                    }
                    else{
                            System.out.println("文件接受地址:D:/");
                        	//保存文件
                            it.write(new File("D:/"+it.getName()));
                    }
                }

        }
```

## 下载

```java
        //获取要下载的文件名
        String filename="list_item.gif";
        //读取要下载的文件内容
        ServletContext servletContext=getServletContext();
        //获取要下载的文件类型
        String mimetype=servletContext.getMimeType("/file/"+filename);
        //通过响应头告诉客户端返回的数据类型
        resp.setContentType(mimetype);
        //告诉客户端用于下载
        resp.setHeader("Content-Disposition","attachment;filename");
        //初始化一个输入流
        InputStream inputStream=servletContext.getResourceAsStream("/file/"+filename);
        //初始化一个输出流
        OutputStream outputStream1=resp.getOutputStream();
        //读取输入流全部内容赋值给输出流,输出给客户端
        IOUtils.copy(inputStream,outputStream1);
```
### 下载名中文乱码问题
#### IE和谷歌浏览器
`resp.setHeader("Content-Disposition","attachment;filename="+URLEncoder.encode("文件名","UTF-8"));`
#### BASE64编码解码
```java
//编码
String content="需要编码的中文";
BASE64Encoder base64encode=new BASE64Encode();
String encodecontent=base64encode.encode(content.getBytes("UTF-8"));
//解码
BASE64Decoder base64decode=new BASE64Decoder();
byte[] data=base64decode.decodeBuffer(encodecontent);
content =new String(data,"UTF-8");
```
#### 火狐浏览器
`resp.setHeader("Content-Disposition","attachment;filename==?UTF-8?Bxxxx?=");`
=? 表示内容开始
UTF-8 表示字符集
B 表示BASE64编码
xxxx 文件名经过BASE64编码后的内容
?= 表示编码内容的结束
#### 动态切换
判断浏览器
`req.getHeader("User=Agent").contain()`

# Cookie
Cookie是一段不超过4KB的小型文本数据，由一个名称（Name）、一个值（Value）和其它几个用于控制Cookie有效期、安全性、使用范围的可选属性组成。

## 创建/获取/修改
```java 
        //创建Cookie
        Cookie cookie =new Cookie("lqw","097");
        response.addCookie(cookie);
        //获取COokie
        System.out.println(Arrays.toString(request.getCookies()));
        //修改Cookie
        //方案1:new 同名cookie 调用addCookie覆盖原Cookie
        //方案2:找到原来Cookie,setValue(),addCookie
```

## 生命控制
setMaxAge(int);
    正数  :  cookie存活时间
    负数  :  浏览器关闭失效
    0  :  立即失效

## 有效路径Path
`cookie.setPath(request.getContextPath()+"/abc");`
Path属性：定义了Web站点上可以访问该Cookie的目录

# Session
1、session就一个接口( HttpSession )。
2、Seszion就是会话。它是用来维护一个客户端和服务器之间关联的一种技术。
3、每个客户端都有自己的一个 session会话。
4、 Session会话中，我们经常用来保存用户登录之后的信息。

## 创建和获取
request.getSession();
    第一次调用是创建
    之后调用时获取创建好的Session会话对象
isNew();
    true 表示刚创建
    false 表示获取之前创建的
getId();
每个会话都有一个唯一的ID值

## 数据域的存取
setAttribute();
getAttribute();

## 生命周期
Tomcat中默认时长:30min
设置单独某个Session超时时长:
```java
setMaxInactiveInterval(int interval);
getMaxInactiveInterval();
```
    正数  :  超时时长时间
    负数  :  永不朝时(极少使用)
    invalidate() 立即超时
设置整个web工程超时时长
```xml
<session-config>
	<session-timeout>时长</session-timeout>
</session-config>
```

# Filter过滤器
Java三大组件之一,javaEE规范,主要作用是拦截请求/过滤响应.

## 实现
1、编写一个类去实现 Filter接口
2、实现过滤方法doFilter()
3、到 web.xml中去配置 Filter的拦截路径
	```java
	    <filter>
        <!--给Filter起一个别名-->
        <filter-name>HelloFilter</filter-name>
        <!--filter完整全类名-->
        <filter-class>HelloFilter</filter-class>
    </filter>
    <!--配置拦截路径-->
    <filter-mapping>
        <!--表示当前路劲给哪个filter使用-->
        <filter-name>HelloFilter</filter-name>
        <url-pattern>/jsptest.jsp</url-pattern>
    </filter-mapping>
    ```

## 生命周期
Filter的生命周期包含几个方法
    1、构造器方法
    2、init初始化方法
        第1，2步，在web工程启动的时候执行（Filter已经创建)
    3、 doFilter过滤方法
        第3步，每次拦截到请求，就会执行
    4、destroy销毁
        第4步，停止web工程的时候，就会执行（停止web工程，也会销毁Filter过滤器)

## FilterConfig
Filter过滤器的配置文件,Tomcat创建Filter的时候自动生成.
FilterConfig 类的作用是获取filter过滤器的配置内容:
    1、获取Filter的名称filter-name 的内容
    2、获取在Filter中配置的 init-param初始化参数
    3、获取ServletContext对象

## FilterChain过滤器链
多个过滤器如何一起工作
FilterChain.doFilter 0方法的作用
    1、执行下一个Filter过滤器(如果有Filter)
    2、执行目标资源（没有Filter)
多个Filter过滤器执行的特点:
    1、所看iter和百标资源双认都执行在同:一个线程中
    2、多个Filter共同执行的时候,它们都使∶用同一个Request对象。
    3、在多个Filter过滤器执行的时候，它们执行的优先顺序是由他们在web.xml中从上到下配置的顺序决定.

## Filter的拦截路径
1. 精确匹配
	`<url-pattern>/jsptest.jsp</url-pattern>`
2. 目录匹配
	`<url-pattern>/web/*</url-pattern>`
3. 后缀名匹配
	`<url-pattern>*.html</url-pattern>`

# JSON
JSON：JavaScript 对象表示法（JavaScript Object Notation）。 JSON 是存储和交换文本信息的语法。类似 XML。 JSON 比 XML 更小、更快，更易解析。

## 定义
value可以是任意数据类型
`var jsonobj{
	"key1":value,
	"key2":value2
}`

## 访问
与访问对象属性一致
`jsonobj.key;`

## 两个常用方法
JSON.stringfy()  把json对象转为json字符串,一般进行数据交换用json字符串
JSON.parse() 把json字符串转为json对象,一般获取数据用json对象

## JSON在java中使用
依赖:gson.jar
1. JavaBean和json转换
	```java
	Book book=new Book("001","hhh","100");
        Gson gson=new Gson();
        //利用gson api将javaBean转为json字符串
        String bookinfo=gson.toJson(book);
        System.out.println(bookinfo);
        //将json字符串转为JavaBean对象
        Book book1=gson.fromJson(bookinfo,Book.class);
        System.out.println(book1);
   ```
2. list集合和json转换
	* 转为json字符串与上相同
	* json字符串转list
	`ArrayList<Book> books=gson.fromJson(booklistinfo,new BookListType().getType());`
	第二个参数为继承TypeToken类的一个类
	`public class BookListType extends TypeToken<ArrayList<Book>> {}`

## map和json转换
	与list相同

# AJAX
AJAX = Asynchronous JavaScript and XML（异步的 JavaScript 和 XML）。
AJAX 不是新的编程语言，而是一种使用现有标准的新方法。
AJAX 最大的优点是在不重新加载整个页面的情况下，可以与服务器交换数据并更新部分网页内容。
AJAX 不需要任何浏览器插件，但需要用户允许JavaScript在浏览器上执行。

## JS原生AJAX请求
```javascript
    // 1. 创建XMLHttpRequest对象
    var ajax=new XMLHttpRequest();
    // 2.调用open方法设置请求参数
    ajax.open("GET","http://localhost:8080/untitled_war_exploded/ajax?action=doGet",true);
    // 3.在send方法前绑定.onreadystatechange事件,处理请求完成后的操作
    ajax.onreadystatechange=function (){
        if(ajax.readyState===4&&ajax.status===200){
            document.getElementById("div1").innerHTML=ajax.responseText;
        }
    }
    // 4.调用send()方法发送请求
    ajax.send();
```

## 同步和异步 async
在Jquery中ajax方法中async用于控制同步和异步，当async值为true时是异步请求，当async值为fase时是同步请求。ajax中async这个属性，用于控制请求数据的方式，默认是true，即默认以异步的方式请求数据。
1、async值为true （异步）
当ajax发送请求后，在等待server端返回的这个过程中，前台会继续 执行ajax块后面的脚本，直到server端返回正确的结果才会去执行success，也就是说这时候执行的是两个线程，ajax块发出请求后一个线程 和ajax块后面的脚本（另一个线程）
2、async值为false （同步）
当执行当前AJAX的时候会停止执行后面的JS代码，直到AJAX执行完毕后时，才能继续执行后面的JS代码。

## JQuery实现ajax
### $.ajaxt方法
	url		请求的地址
	type	请求的类型GET/POST
	data	发送给服务器的数据,两种格式
				1. name1=valu1e & name2=value2;
				2. {key1:value1}
	success	请求成功响应的回调函数
	dataType  响应的数据类型

### $.get 和$.post

### $.getJSON
	url		请求的地址
	data	发送给服务器的数据
	callback   回调函数

### 表单序列化serialize()
可以把表单中所有表单项内容获取到,并以"name=value"的形式拼接

# i18n国际化
## 国际化三要素
    * Locale对象:表示不同的时区 位置 语言
    * Properties属性配置文件:i18n_zh_CN.properties
    * ResourceBundle资源包:根据Locale和Properties获取文字信息

# MAVEN
## Maven作用
项目自动化构建工具
1. 项目的自动构建,帮助开发人员做项目代码的编译,测试,打包,安装,部署等工作.
2. 管理依赖
## 约定的目录结构
``` 
	项目名
		\src
			\mian                                           主程序目录
				\java                                   源代码
				\resources                         配置文件
			\test                                             测试程序代码
				\java                                  测试代码的(junit)
				\resources                        测试程序需要的配置文件
		\pom.xml                                            maven的配置文件(核心文件)
```
## POM
Project Object Model项目对象模型,pom.xml 就是 maven 的配置文件，用以描述项目的各种信息。
### 坐标

https://mvnrepository.com/

``` xml
<!-- project时根标签,后面是约束文件-->
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <!--pom模型版本-->
  <modelVersion>4.0.0</modelVersion>
  <!--坐标gav-->
  <groupId>org.codehaus.mojo</groupId><!--组织名称-->
  <artifactId>my-project</artifactId><!--项目名称-->
  <version>1.0</version><!--项目版本号-->
  <!--项目打包类型-->
  <packaging>war</packaging>
</project>

```

### 依赖
``` xml
<dependencies>
    <dependency>
        gac
    </dependency>
</dependencies>
```

## 仓库
仓库是存东西的,maven的仓库存放的是:
1. maven工具自己的jar包。
2. 第三方的其他jar，比如项目中要使用mysgl驱动。
3. 自己写的程序，可以打包为jar。存放到仓库。
仓库的分类:
1.本地仓库(本机仓库)︰位于你自己的计算机，它是磁盘中的某个目录。本地仓库:默认路径，是你登录操作系统的账号的目录中/.m2/repository
c : \users \NING MEI\.m2(repository
修改本地仓库的位置:修改maven工具的配置文件(maven的安装路径\conflsetting.xml)
步骤:
1）创建一个目录，作为仓库使用。目录不要有中文和空格。目录不要太深。
例如:D:lopenrepository
2）修改setting.xml文件，指定 D:lopenrepository这个目录
`<localRepository>D :/ openrepository</localRepository>`

## 生命周期/插件/命令
maven的生命周期:项目构建的各个阶段。包括清理，编译，测试，报告，打包，安装，部署
插件:要完成构建项目的各个阶段，要使用maven的命令，执行命令的功能是通过插件完成的。插件就是jar，一些类。
命令:执行maven功能是由命令发出的。比和mvn compile

## 常用命令
（1）maven clean。
对项目进行清理，清理的过程中会删除删除target目录下编译的内容。
（2）maven validate
验证，验证项目是正确的并且所有的信息是可用的；
（3）maven compile。
编译项目源代码。
（4）maven test。
对项目的运行测试。
（5）maven packet。
可以打包后的文件存放到项目的 target 目录下，打包好的文件通常都是编译后生成的class文件。
（6）verify
运行任何检查，验证包是否有效且达到质量标准。
（7）maven install。
在本地仓库生成仓库的安装包可以供其他项目引用，同时打包后的文件存放到项目的 target 目录下。
（8）site
 生成项目相关信息的网站 
（9）deploy
部署，在构建环境中完成，复制最终的包到远程库。

## 依赖管理
依赖范围:使用scope表示依赖范围
	compile:默认,参与项目构建的所有阶段
	test:测试,在测试阶段使用
	provided:提供者,项目在部署到服务器时,不需要提供这个依赖的jar,而是由服务器这提供这个依赖
## 常用设置
### properties配置
```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding><!--项目构建编码-->
    <maven.compiler.source>1.7</maven.compiler.source><!--源码编译jdk版本-->
    <maven.compiler.target>1.7</maven.compiler.target><!--运行代码的jdk版本-->
  </properties>
```
### 自定义变量
1. 定义
    <properties>
    <变量名>变量值</变量名>
    </properties>
2. 使用
    <dependency>
    <version>${变量名}</version>
    </dependency>