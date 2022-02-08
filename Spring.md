# Spring 概述
1. Spring 是轻量级的开源的JavaEE框架
2. Spring可以解决企业应用开发的复杂性
3. 两个核心部分:IOC和Aop
   (1)IOC:控制反转,把创建对象过程交给Spring进行管理
   (2)Aop:面向切面,不修改源代码进行功能管理
4. Spring特点:
   (1)方便解耦,简化开发
   (2)Aop编程支持
   (3)方便编程测试
   (4)方便和其他框架整合
   (5)方便事务管理
   (6)降低API开发难度
# IOC容器
依赖:
   commons-logging.jar
   spring-beans.RELEASE.jar
   spring-core.RELEASE.jar
   spring-context.RELEASE.jar
   spring-expression.RELEASE.jar

控制反转（Inversion of Control），是面向对象编程中的一种设计原则，可以减低耦合度。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体将其所依赖的对象的引用传递给它。

## 底层原理
1. xml解析
   配置xml文件,配置你创建的对象
2. 工厂模式
   有service类和和dao类,创建工厂类
3. 反射
   创建对象类
## IOC接口
Spring提供IOC容器实现的两种方式(两个接口):
1. BeanFactor : IOC容器基本实现,是Spring内部的使用接口,不提供开发人员使用. BeanFactor加载配置文件的时候不会创建对象,在获取对象的时候才会创建.
2. ApplicationContext : BeanFacotor接口的子接口,一般由开发人员使用. ApplicationContext加载配置文件的时候会就会创建对象.
ApplicationContext两个实现类:
* FIleSystemXmlApplicationContext
  默认文件路径是src下那一级
    classpath:和classpath\*:的区别:
    classpath: 只能加载一个配置文件,如果配置了多个,则只加载第一个
    classpath\*: 可以加载多个配置文件
* ClassPathXmlApplicationContext
    默认获取的是项目路径,默认文件路径是项目名下一级，与src同级。
    如果前边加了file:则说明后边的路径就要绝对路径

## IOC操作:Bean管理

### 什么是Bean管理
1. 创建对象
2. 注入属性

### Bean管理两种实现方式

#### 1. 基于xml配置文件

##### 创建对象
`<bean id="对象标识",class="类完整路径"></bean>`
默认调用无参构造创建对象
bean标签中常用属性:
    id : 对象唯一标识
    class : 类全类名

##### 注入属性
DI : 依赖注入
###### (1)使用set方法注入
```xml
<!--类的java文件中要有该属性的set方法-->
<bean id="对象标识",class="类完整路径">
	<property name="属性名" value="属性值"</property>
</bean>
```
###### (2)使用有参构造注入
```xml
<bean id="对象标识",class="类完整路径">
	<constructor-arg name="属性1" value="属性值"></constructor-arg>
	<constructor-arg name="属性2" value="属性值"></constructor-arg>
</bean>
```

###### (3) p名称空间注入

###### (4)注入空值和特殊符号
```xml
<!--注入空值-->
<property name="属性名">
	<null/>
</property>
<!--注入特殊符号
1. 转义符
2. CDATA 
-->
<property name="属性名">
	<value>![CDATA[属性值]]</value>
</property>
```

###### (5)注入外部Bean
```xml
<bean id="..." class="...">
	<property> name="属性名" ref="外部bean标签id"</property>
</bean>
<bean id="外部bean标签id" class="..."> </bean>
```

###### (6) 内部bean和级联赋值
```xml
<!--内部bean-->
<bean id="..." class="...">
	<property> name="属性名">
		<bean ...>...</bean>
	</property>
</bean>
<!--级联赋值
级联赋值需要get和set方法
-->
<bean id="..." class="...">
	<property> name="属性名.子属性名" value="属性值"></property>
</bean>
```

###### (7)集合注入
```xml
<bean id="l1" class="List1">
        <!--数组注入-->
        <property name="array">
            <array>
                <value>1</value>
                <value>2</value>
            </array>
        </property>
        <!--list注入-->
        <property name="list">
            <list>
                <value>a</value>
                <value>b</value>
            </list>
        </property>
        <!--map注入-->
        <property name="map">
            <map>
                <entry key="id" value="1"> </entry>
            </map>
        </property>
        <!--set注入-->
        <property name="set">
            <set>
                <ref bean="b1"/>
                <ref bean="b2"/>
            </set>
        </property>
    </bean>
```

提取集合注入部分
```xml
<!--引入uitl名称空间-->
xmlns:util="http://www.springframework.org/schema/util"
xsi:schemaLocation="
http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd">
<util:list id="format_list">
    <value>a</value>
    <value>b</value>
</util:list>
```

##### 工厂Bean
在配置文件定义的bean类型可以和返回类型不一样
第一步 创建类,让这个类作为工厂bean,实现接口FactoryBean
第二部 实现接口里面的方法,在实现的方法中定义返回的bean类型

```Java
public class MyBean implements FactoryBean<Book> {
    @Override
    public Book getObject() throws Exception {
        Book book=new Book();
        return book;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }

    @Override
    public boolean isSingleton() {
        return FactoryBean.super.isSingleton();
    }
}
```

##### Bean作用域
1. singleton：单例模式，Spring IoC容器中只会存在一个共享的Bean实例，无论有多少个Bean引用它，始终指向同一对象。Singleton作用域是Spring中的缺省作用域，也可以显示的将Bean定义为singleton模式，配置为：
    `<bean id="userDao" class="com.ioc.UserDaoImpl" scope="singleton"/>`

2. prototype:原型模式，每次通过Spring容器获取prototype定义的bean时，容器都将创建一个新的Bean实例，每个Bean实例都有自己的属性和状态，而singleton全局只有一个对象。根据经验，对有状态的bean使用prototype作用域，而对无状态的bean使用singleton作用域。

3. request：在一次Http请求中，容器会返回该Bean的同一实例。而对不同的Http请求则会产生新的Bean，而且该bean仅在当前Http Request内有效。
    `<bean id="loginAction" class="com.cnblogs.Login" scope="request"/>`,针对每一次Http请求，Spring容器根据该bean的定义创建一个全新的实例，且该实例仅在当前Http请求内有效，而其它请求无法看到当前请求中状态的变化，当当前Http请求结束，该bean实例也将会被销毁。

4. session：在一次Http Session中，容器会返回该Bean的同一实例。而对不同的Session请求则会创建新的实例，该bean实例仅在当前Session内有效。
    `<bean id="userPreference" class="com.ioc.UserPreference" scope="session"/>`,同Http请求相同，每一次session请求创建新的实例，而不同的实例之间不共享属性，且实例仅在自己的session请求内有效，请求结束，则实例将被销毁。

5. global Session：在一个全局的Http Session中，容器会返回该Bean的同一个实例，仅在使用portlet context时有效。

##### Bean生命周期
1. 通过构造器创建bean实例（无参数构造)。
2. 为bean的属性设置值和对其他bean引用（调用set方法)
3. 把bean实例传给后置处理器postProcessBeforeInitialization方法
4. 调用bean的初始化的方法（需要进行配置初始化的方法)
	`<bean init-method="Init">`
5. 把bean实例传给后置处理器postProcessAfterInitialization方法
	后置处理器会处理每一个Bean对象
	后置处理器使用方法:
	(1)定义一个类实现接口BeanPostProcessor
	(2)重写 postProcessBeforeInitialization/postProcessAfterInitialization 方法
	(3)配置xml `<bean id="myProcess" class="全类名"> </bean>`
6. bean可以使用了(对象获取到了)
7. 当容器关闭时候，调用bean的销毁的方法（需要进行配置销毁的方法)
	`<bean destroy-method="destroy">`

##### 测试类
```java
//加载配置文件
        ApplicationContext applicationContext=new ClassPathXmlApplicationContext("spring.xml");
```

##### xml自动装配
根据指定装配规则(属性名或者属性类型),Spring自动将匹配的属性注入.
	`<bean autowire="byName/byType">`
   byName : 根据属性名注入,注入的bean id要和类属性名一样
   byTtype : 根据属性类型注入

```xml
<beans
xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
```

2. 编写配置文件 data.properties
3. 引入配置文件
`<context:property-placeholder location="data.properties"/>`
4.使用${key}表达式注入属性
`<property name="id" value="${id}"> </property>` 


#### 2. 基于注解
依赖 : spring-aop.RELEASE.jar
1. 注解是代码特殊标记,格式: @注解名称(属性名称=属性值)
2. 注解作用在 类/方法/属性 上面
3. 使用注解目的:简化xml配置
##### 对象创建
1. 常用创建对象注解 
    @Component 业务特殊组件层,如handler类
    @Controller 业务控制层
    @Service 业务逻辑层
    @Repository 业务资源层
    *四个注解都可以用于创建Bean实例,一般来说不同场景使用不同注解
2. 创建对象步骤
    (1)引入依赖:spring-aop.RELEASE.jar ,引入命名空间:context
    (2)开启组件扫描
    ```xml
    <!--开启组件扫描
        扫描多个包可用逗号隔开
    -->
    <context:component-scan base-package="包名"> </context:component-scan>
    ```
     组件扫描中的细节问题:
    ```xml
     <!--组件扫描细节 示例一
     use-default-filters="false" 设置不使用默认扫描
     <context:include-filter> 设置组件扫描的目标内容
    -->
    <context:component-scan base-package="com.lqw" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Component"/>
    </context:component-scan>
    <!--组件扫描细节 实例二
    <context:exclude-filter> 设置组件扫描不扫描的内容
    -->
    <context:component-scan base-package="com.lqw">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    ```
    (3)在类上添加创建对象注解
    `//value为实例名称,可省略,默认为类名首字母小写
    @Component(value = "p1")`

##### 属性注入
1. 常用属性注入注解
@Autowired : 根据属性类型进行自动装配
@Qualifier : 根据属性名称进行注入,
@Resource : 可以根据类型注入，可以根据名称注入·
@Value : 注入普通类型属性。
2. 属性注入步骤
    (1)创建类对象
    (2)使用注解注入属性
##### 完全注解开发
1. 创建配置类(替代xml配置文件)

`
@Configuration //标记作为配置类
@ComponentScan(basePackages = "com.lqw") //开启组件扫描
@EnableAspectJAutoProxy(proxyTargetClass=true) //开启生成代理对象
`

2. 编写测试类
`ApplicationContext applicationContext=new AnnotationConfigApplicationContext(配置类.class);`

# AOP
* 面向切面编程（方面)，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
* 通俗描述:不通过修改源代码方式，在主千功能里面添加新功能

## 底层原理
AOP底层使用动态代理,有两种情况
1. 有接口情况,使用JDK动态代理,创建接口实现类的代理对象,增强类的方法.
代码实现
```Java
static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) ;
```

返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。 
第一参数 : 类加载器
第二参数 :  增强方法所在的类，这个类实现的接口，支持多个接口
第三参数 :  实现这个接口InvocationHandler，创建代理对象，写增强的方法
(1)创建接口,定义方法
```Java
public interface aopInterface {
   public int add(int a,int b);
}
```
(2)创建接口实现类,实现方法
```Java
public class JdkProxy {
    public static void main(String[] args) {
        //创建接口实现代理对象
        Class[] interfaces = {aopInterface.class};
        aopClass aop=new aopClass();
        aopInterface aopInterface=(aopInterface)Proxy.newProxyInstance(JdkProxy.class.getClassLoader(), interfaces, new aopProxy(aop));
        int res=aopInterface.add(1,2);
        System.out.println(res);
    }
}
class aopProxy implements InvocationHandler {
    private Object obj;
    //传入代理对象源对象
    public aopProxy(Object obj){
        this.obj=obj;

    }
    //增强的逻辑
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before");
        Object res=method.invoke(obj,args);
        System.out.println("after");
        return res;
    }
} 
public class aopClass implements aopInterface{
    @Override
    public int add(int a,int b) {
        return a+b;
    }
} 
```

(3)使用Proxy类创建接口代理对象
2. 无接口情况,使用CGLIB动态代理,创建当前类子类的代理对象,增强类的方法.
##  术语
1. 连接点 
类里面可以被增强的方法
2. 切入点
实际被增强的方法
3. 通知(增强)
	(1)实际增强的逻辑部分称为通知(增强)
	(2) 通知类型
	* 前置通知
	* 后置通知
	* 环绕通知
	* 异常通知
	* 最终通知
4. 切面
把通知应用到切入点的过程

## 基于AspectJ实现AOP操作
加入依赖: 
	spring-aspects-5.2.6.RELEASE.jar
	com.springsource.org.aspectj.weaver-1.7.2.RELEASE.jar
	com.springsource.org.aopalliance-1.0.0.jar
	com.springsource.net.sf.cglib-2.2.0.jar
AspectJ是独立于Spring的AOP框架,一般把spring和AspectJ结合使用进行AOP操作.

### 切入点表达式
作用: 知道对哪个类里面的哪个方法进行增强
execution(\[权限修饰符]\[返回类型]\[类全路径]\[方法名称]([参数列表]))

### 基于注解
需要名称空间:aop/context
步骤:
1. 创建委托类
```Java
@Component(value = "d")
public class Delegate {
    public int add(int a,int b){
        System.out.println("add ");
        return a+b;
    }
}
```

2. 创建增强类(增强逻辑)
在增强类里创建方法,让不同方法代表不同通知类型.
```java
@Component(value = "p")
@Aspect //生成代理对象
public class DelegateProxy {
    @Before(value = "execution(* aop.Delegate.add(..))")
    public void before(){
        System.out.println("before....");
    }

    @After(value = "execution(* aop.Delegate.add(..))")
    public void after(){
        System.out.println("after...");
    }

}
```

3. 进行通知配置
(1)开启注解扫描
`<context:component-scan base-package="aop"> </context:component-scan>`
(2)使用注解创建委托类和增强类对象
(3)开启生成代理对象
`<aop:aspectj-autoproxy> </aop:aspectj-autoproxy>`

4. 配置不同类型通知
在增强类上方法上使用注解
@Before: 前置通知, 在方法执行之前执行
@After: 后置通知, 在方法执行之后执行 。
@AfterRunning: 返回通知, 在方法返回结果之后执行
@AfterThrowing: 异常通知, 在方法抛出异常之后
@Around: 环绕通知, 围绕着方法执行

5. 公共切入点抽取
定义方法使用@pointcut 注解抽取公共切入点
`@Pointcut(value = "execution(* aop.Delegate.add(..))")
    public void pointSet(){
    }`
使用公共切入点
`@Before(value="pointSet()");`

6. 设置增强类优先级
在增强类上添加注解@Order( int priority) 值越小优先级越高

### 基于配置文件
1. 创建委托类和增强类

2. 在配置文件中创建两个类的对象

3. 在配置文件中配置切入点

   ```xml
   <!--aop配置增强-->
   <aop:config>
       <!-切入点--->
   	<aop:pointcut id="p" expression="execution(...)"/>
       <!--配置切面-->
       <aop:aspect ref="增强类对象">
           <!--zengq方法具体作用的方法-->
       <aop:before method="增强方法名" pointcut-ref="p" />
   </aop:config>
   ```

# jdbcTemplate
## 准备工作
1. 添加依赖:
   	druid-1.1.10.jar
   	mysql-connector-java-8.0.16.jar
   	spring-tx-5.2.6.RELEASE.jar
   	spring-orm-5.2.6.RELEASE.jar
   	spring-jdbc-5.2.6.RELEASE.jar
   spring框架对jdbc进行封装,方便使用sprint操作数据库 
2. 配置数据库连接池
```xml
<bean id="database" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close" >
        <property name="url" value="jdbc:mysql:///jdbctest.db" />
        <property name="name" value="root" />
        <property name="password" value="123465" />
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
</bean>
```
3. 创建jdbcTemplate对象,注入dataSource
```xml
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
```
4. 创建service类,创建dao类. 在dao注入jdbcTemplate对象
```Java
@Repository
public class HelloDaoImpl implements HelloDao{
    //注入jdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public void add(Person person) {
    }
}
```
在service注入dodao对象
```Java
@Service
public class HelloService {
    //注入Dao
    @Autowired
    private HelloDaoImpl helloDao;

    public void addPerson(Person person){
        helloDao.add(person);
    }
}

```

## 操作数据库
### 添加
```Java
String sql="insert into person values(?,?)";
jdbcTemplate.update(sql,person.getId(),person.getName());
```
### 修改和删除
与添加一致调用update方法

### 查询
1.  返回某个值
使用方法:
`queryForObject(String sql, Class<T> requiredType)`
实现代码:
```java
String sql="select count(*) from person";
return jdbcTemplate.queryForObject(sql,Integer.class);
```

2.  返回对象
使用方法:
`queryForObject(String sql, RowMapper<T> rowMapper, @Nullable Object... args)`
第二个参数RowMapper是个是个接口,针对不同数据类型,使用该接口的实现类实现对不同数据类型的封装.
代码实现:
```Java
String sql="select * from person where id=?";
//该对象类型一定要有无参构造
Person person= jdbcTemplate.queryForObject(sql,new BeanPropertyRowMapper<Person>(Person.class),id);
```

3.  返回集合
使用方法:
`query(String sql, RowMapper<T> rowMapper)`
实现代码:
```java
String sql="select * from person";
List<Person> personList =jdbcTemplate.query(sql,new BeanPropertyRowMapper<Person>(Person.class));
```

### 批量操作
1. 批量添加
使用方法:
`batchUpdate(String sql, List<Object[]> batchArgs)`
代码实现:
```java
public void addPersons(List<Object[]> data) {
		//执行逻辑是 将data中每一个数据行执行一遍sql语句
        String sql="insert into person values(?,?)";
        jdbcTemplate.batchUpdate(sql, data);
    }
```
```java
        System.out.println(helloService.selectInfo(2));
        System.out.println(helloService.selectAll());
        System.out.println(helloService.selectCount());
        List<Object[]> data=new ArrayList<>();
        Object[] o1={102,"liu"};
        Object[] o2={103,"wang"};
        data.add(o1);
        data.add(o2);
        helloService.addPerson(data);
```
2. 批量修改/批量删除
与批量添加一致

# 事务管理
* 事务管理一般添加到Service层

## 编程式事务管理

## 声明式事务管理(常用)

### 基于注解
需要名称空间context/aop/tx
1.  在xml配置文件中配置事务管理器
```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"> 
        <!--注入数据源-->
        <property name="dataSource" ref="dataSourceBank"> </property>
    </bean>
```
2. 开始事务注解
```xml
<tx:annotation-driven transaction-manager="transactionManager"/>
```
3. 在Service类或方法上加事务注解
	@Transactional[(参数)]
	propagation : 事务传播行为
        REQUIRED：支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。 
        SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行。 
        MANDATORY：支持当前事务，如果当前没有事务，就抛出异常。 
        REQUIRES_NEW：新建事务，如果当前存在事务，把当前事务挂起。 
        NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 
        NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。 
        NESTED：支持当前事务，如果当前事务存在，则执行一个嵌套事务，如果当前没有事务，就新建一个事务。
	
	ioslation : 隔离级别
		READ_UNCOMMITTED:读未提交
		READ_COMMITED:读已提交
		REPEATABLE_READ:可重复读
		SERLALIZABLE:串行化
	timeout : 超时时间,默认值-1 以秒为单位
	readOnly : 是否只读 
	rollbackFor : 回滚
	noRollbackFor : 不回滚
		设置哪些异常需要回滚

### 基于xml配置文件
1. 配置事务管理器

2. 配置通知

```xml
    <tx:advice id="txadvice">
        <tx:attributes>
            <!--选择一种或一类方法-->
            <tx:method name="transfer" propagation="REQUIRED"/>
            <!--   <tx:method name="teansfer*" propagation="REQUIRED"/>  -->
        </tx:attributes>
    </tx:advice>
```

3. 配置切入点和切面

### 完全注解开发
```java
@Configuration //配置类
@ComponentScan(basePackages = "transaction") //开启组件扫描
@EnableTransactionManagement //开启事务

public class Txconfig {
    //创建数据连接池
    @Bean
    public DruidDataSource getDruidDataSource(){
        DruidDataSource dataSource=new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");//注册驱动
        dataSource.setUrl("jdbc:mysql://127.0.0.1:3306/jdbctest?serverTimezone=UTC");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        return dataSource;
    }
    
    //创建jdbcTemplate对象
    @Bean
    public JdbcTemplate getJdbcTemplate(DruidDataSource dataSource){
        //到ioc容器中根据类型找到dataSource
        JdbcTemplate jdbcTemplate=new JdbcTemplate();
        //注入dataSource
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }
    //创建事务管理器
    @Bean
    public DataSourceTransactionManager getDataSourceTransationManager(DruidDataSource dataSource){
        DataSourceTransactionManager manager=new DataSourceTransactionManager();
        manager.setDataSource(dataSource);
        return manager;
        
    }
}
```

# spring5新功能
spring4全框架基于java8

##  整合日志框架
Spring5自带通用日志框架;
Spring5 移除LogjConfigListener,官方建议使用Log4j2;
Log4j2使用:
	1. 依赖:
		log4j-api-2.14.1.jar
		log4j-core-2.14.1.jar
		log4j-to-slf4j-2.14.1.jar
		slf4j-api-1.7.25.jar
	2. 创建log4j2.xml配置文件
```xml 
	<?xml version="1.0" encoding="UTF-8"?>
<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出-->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
<Configuration status="ALL">
    <!--定义所有的appender-->
    <Appenders>
        <!--输出日志信息到控制台-->
        <Console name="Console" target="SYSTEM_OUT">
            <!--输出日志信息的格式-->
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>
    <!--定义logger,并引入appender,appender才会生效-->
    <Loggers>
        <!--root:用于指定项目根日志,如果没有单独指定logger,则会使用root作为默认的日志输出-->
        <Root level="all">
            <appender-ref ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```


## @Nullable注解
1. 使用在方法上,方法放回值可以为空
2. 使用在方法参数里面,方法参数可以为空
3. 使用在属性上,属性值可以为空

## 函数式风格 GenericApplicationContext
```java
    public void test01(){
        //创建GenericApplicationContext对象
        GenericApplicationContext context=new GenericApplicationContext();
        //清空内容
        context.refresh();
        //注册对象
        context.registerBean("book10",Book.class, Book::new);
        //获取对象
        Book book = (Book) context.getBean("book10");
        System.out.println(book);
    }
```

## 单元测试框架JUnit5
### 整合JUnit4
1. 依赖
	spring-test-5.2.6.RELEASE.jar
2. 创建注册类,使用注解方式完成
```Java 
@RunWith(SpringJUnit4ClassRunner.class) //单元测试框架版本
@ContextConfiguration("classpath:spring.xml") //加载配置文件
public class TestJUnit4 {
    //自动获取对象并装填
    @Autowired
    private object1 o1;
    @Test
    public void test01(){
        System.out.println(o1);
    }
}
```

### 整合JUnit5
```java 
@ExtendWith(SpringExtension.class)
@ContextConfiguration("classpath:spring.xml")
public class TestJUnit5{
    @Autowired
    private object1 o1;

    @Test
    public void test01(){
        System.out.println(o1);
    }
}
```

## SpringWebFlux

### 简介
用于Web开发, 功能类似SpringMVC, 使用响应式编程框架.





