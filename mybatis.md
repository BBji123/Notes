# 框架概述
## 三层架构
1. 界面层(视图层) UI
主要是指与用户交互的界面。用于接收用户输入的数据和显示处理后用户需要的数据。
2. 业务逻辑层 BLL
UI层和DAL层之间的桥梁。实现业务逻辑。业务逻辑具体包含：验证、计算、业务规则等等。
3. 持久层(数据访问层) DAL
与数据库打交道。主要实现对数据的增、删、改、查。将存储在数据库中的数据提交给业务层，同时将业务层处理的数据保存到数据库。（当然这些操作都是基于UI层的。用户的需求反映给界面（UI），UI反映给BLL，BLL反映给DAL，DAL进行数据的操作，操作后再一一返回，直到将用户所需数据反馈给用户）

## 原生JDBC
### 回顾编程六步
1. 注册驱动
2. 获取链接
3. 获取数据库执行对象
4. 执行SQL语句
5. 处理数据集
6. 关闭资源

### 缺点
1. 数据库链接频繁创建/释放,浪费资源,影响性能.
2. sql语句和业务逻辑层代码混在一起,编写sql需要考虑占位符一一对应,导致sql编写困难,且维护困难.
3. 需要手动解析结果集.

## MyBatis框架
* 持久层框架, 原名ibatis, 3.0后改名MyBatis. 
* MyBatis功能:
    1) 注册驱动
    2) 创建JDBC中需要的Connection/Statement/ResultSet
    3) 执行sql语句,得到ResultSet
    4) 处理ResultRet,把记录集中的数据封装成pojo对象,同时将其放入List集合
    5) 关闭资源
    6) 实现Sql语句和java代码的解耦合

# Mybatis入门

## 第一个Mybatis程序

### 1. 创建数据表
### 2. 新建maven项目
   1. 加入mybatis依赖,mysql驱动,junit

```xml
    <properties>
        <maven.compiler.source>15</maven.compiler.source>
        <maven.compiler.target>15</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.7</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.26</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```
   2. 在\<build> 中加入资源插件

```xml
    <build>
        <!--资源插件 处理xml文件-->
        <resources>
            <resource>
                <directory>src/main/java</directory><!--所在的目录-->
                <includes><!--包括目录下的.properties .xml文件都会被扫描-->
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>
```
### 3. 创建实体类,定义属性,属性名和列名保持一致

```Java
//以Student为例
public class Student {
    private int id;
    private String name;
    private String sex;
    private int age;
}
```
### 4. 创建Dao接口,定义操作数据库方法
### 5. 创建xml文件(mapper文件),写sql语句
	* mybatis推荐sql语句和java代码分开
	* mapper文件:定义和dao接口在同一个目录,一个表一个mapper文件
	StudentDao.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--
    1. 约束文件
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd"
    定义和限制当前文件中可以使用的标签和属性,以及标签的出现顺序
    2. mapper是根标签
    namespace : 命名空间,不能为空,唯一值 作用:参与识别sql语句
    3. mapper里面可以写:<insert> <update> <delete> <select>等标签
    <insert>里面是insert语句,表示执行insert操作
    ...

-->
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
    <select id="selectBlog" resultType="Blog">
        select * from Blog where id = #{id}
    </select>
    <!--查询一个学生
    id : 表示要执行的sql语句的唯一标识,是一个自定义的字符串,推荐使用dao接口中的方法名
    resultType :(java对象的全类名)把sql语句的结果数据赋值给哪个java对象
    -->
    <select id="getById" resultType="com.lqw.mybatis.Bean.Stuodent">
        select * from Students where id=1001
    </select>
</mapper>
```
### 6. 创建mybatis主配置文件,放在resources目录下
   1. 定义创建连接实例的数据源(DataSource)对象
   2. 指定其他的mapper文件位置
      mybatis.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <!--配置数据源 创建Connection对象-->
            <dataSource type="POOLED">
                <!--注册驱动-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/jdbctest?useUnicode=true&amp;
                characterEncoding=utf-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <!--
    指定其他mapper文件的位置
    使用其中的sql语句
    -->
    <mappers>
        <!--
        使用mapper resource属性指定文件路径
        这个路径从target/classes开启
        一个mapper resource 指定一个mapper文件
        -->
        <mapper resource="com/lqw/mybatis/Dao/StudentDao.xml"/>
    </mappers>
</configuration>
```
### 7. 创建测试内容
```Java
public class MyTest {
    //测试mybatis执行sql语句
    @Test
    public void testGetById(){
        //调用mybatis某个对象的方法, 执行mapper文件中的sql语句
        //mybatis核心类 : SqlSessionFactory
        //1. 定义mybatis核心配置文件的位置
        String config="mybatis.xml";
        //2. 读取主配置文件,使用mybatis框架中的Resources类
        InputStream inputStream= null;
        try {
            inputStream = Resources.getResourceAsStream(config);
        } catch (IOException e) {
            e.printStackTrace();
        }
        //3. 创建SqlSessionFactory对象,使用SqlSessionFactoryBuilder类
        SqlSessionFactory factory=new SqlSessionFactoryBuilder().build(inputStream);
        //4. 获取SqlSession对象
        SqlSession session=factory.openSession();
        //5. 指定要执行的sql语句的id
        //id = namespace + 标签id值
        String sqlId="com.lqw.mybatis.Dao.StudentDao"+"."+"getById";
        //6. 通过SqlSession的方法 执行sql语句
        Student student=session.selectOne(sqlId);
        System.out.println(student);
        //7. 关闭资源
        session.close();
    }
}
```

## 占位符
占位符用法
```xml
    <select id="getById1" resultType="com.lqw.mybatis.Bean.Student">
        select * from students where id=#{id}
    </select>
```
调用方法
```Java
        String sqlId1="com.lqw.mybatis.Dao.StudentDao"+"."+"getById1";
        Student student1=session.selectOne(sqlId1,1002);
        System.out.println(student1);
```

## 日志

详细信息查mybatis文档

```xml
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
```

## insert和事务提交
mybatis默认为手动提交事务
`session.commit();`
```xml
    <!--
    如果传给mybatis的是一个Java对象,使用#{属性名}来获取属性值-->
    <insert id="add">
        insert into students values(#{id},#{name},#{sex},#{age})
    </insert>
```

```Java
//返回值为受影响行数
        int times=session.insert(sqlId,new Student(1004,"mx","female",30));
```

## MyBatis中的重要对象
1. Resources : 读取主配置信息
```Java
String config="mybatis.xml";
InputStream inputStream = Resources.getResourceAsStream(config);
```

2. SqlSessionFactoryBuilder : 负责创建SqlSessionFactor对象

3. SqlSessionFactory
   重量级对象 : 创建此对象需要使用更多的资源和时间. 在项目中有一个就可以了
   SqlSessionFactory接口 : SqlSession的工厂,作用就是创建SqlSession.
   SqlSessionFactory中的方法:
   openSession():获取一个默认的SqlSession对象默认是需要手工提交事务的。
   openSession(boolean): boolean参数表示是否自动提交事务。
      true: 创建一个自动提交事务的SqlSession
      false:等同于没有参数的openSession

4. SqlSession
   SqlSession对象是通过SqlSessionFactory获取的。SqlSession本身是接口
   DefaultSqlSession : 实现类
   `public class Defau1tsq1session implements sqlsession { }`
   SqlSession作用是提供了大量的执行sql语句的方法:
    selectone:执行sq1语句，最多得到一行记录，多余1行是错误。selectList:执行sql语句，返回多行数据
    selectMap: 执行sq1语句的，得到一个Map结果insert:执行insert语句
    update:执行update语句delete:执行delete语句commit:提交事务
    ro11back :回顾事务

注意SqlSession对象不是线程安全的，使用的步骤:
    ①:在方法的内部，执行sql语句之前，先获取SqlSession对象
    ②:调用SqlSession的方法，执行sql语句
    ③:关闭SqlSession对象，执行SqlSession.close()

## 创建工具类
由于SqlSessionFactory对象只需要创建一次,所以使用工具类获取该对象.
```Java
public class MybatisTool {
    private static SqlSessionFactory factory=null;
    static{
        String config="mybatis.xml";
        InputStream inputStream= null;
        try {
            inputStream = Resources.getResourceAsStream(config);
            factory=new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    public static SqlSession getSqlSession(){
        SqlSession session=null;
        if(factory!=null){
            session=factory.openSession();
        }
        return session;
    }
}
```

利用工具类实现Dao类的实现类

```Java
public class StudentDaoImpl implements StudentDao{
    @Override
    public Student getById(int id) {
        SqlSession session= MybatisTool.getSqlSession();
        String sql="com.lqw.mybatis.Dao.StudentDao"+"."+"getById1";
        Student student=session.selectOne(sql,id);
        //System.out.println(student);
        return student;
    }
}
```

# dao代理

## Dao实现操作数据库

* mybatis提供代理:
mybatis创建Dao接口的实现类对象，完成对sql语句的执行。mybatis创建一个对象代替你的dao实现类功能。
* 使用mybatis代理要求
   * mapper文件中的namespace一定dao接口的全限定名称
   * mapper文件中标签的id是dao接口方法名称
* mybatis代理实现方式使用SqlSession对象的方法getMapper(dao.class)
* 例如:现在有StudentDao接口。
```Java
sq1session session = MyBatisutils.getsq1session();
studentDao dao = session.getMapper (studentDao.class);
student student = dao.selectById(1001);

//上面代码中
studentDao dao = session. getMapper(studentDao.class);
等同于
studlentDao dao= new studentDaoImp1();
```

使用dao代理,就可以不用实现Dao接口
```Java
    @Test
    public void testProxy(){
        int id=1002;
        SqlSession session= MybatisTool.getSqlSession();
        //获取代理
        StudentDao dao=session.getMapper(StudentDao.class);
        Student student=dao.getById1(id);
        System.out.println(student);
    }
```

## 理解参数
理解参数是:通过java程序把数据传入到mapper文件中的sql语句。参数主要是指dao接口方法的形参

### parameterType
parameterType:表示参数的类型，指定dao方法的形参数据类型。这个形参的数据类型是给mybatis使用。mybatis在给sql语句的参数赋值时使用。PreparedStatement.setXXX(位置，值)

parameterType:指定dao接口形参的类型
这个属性的值可以使用java类型的全限定名称或者 mybatis定义的别名(别名查询MyBatis文档)
mybatis执行的sql语句: select id, name , email,age from student where id=? 
?是占位符，使用jdbc中的Preparedstatement执行这样的sql语句
PreparedStatement pst = conn.preparedStatement("select id,name,email ,age from student where id=?");
给?位置赋值
参数Integer ,执行pst.setInt(1,1005);
参数是string，执行pst.setstring(1, "1005");


```xml
    <select id="getById1" resultType="com.lqw.mybatis.Bean.Student" parameterType="java.lang.Integer">
        select * from students where id=#{id}
    </select>
```

### 接收多个简单类型参数
@Param 命名参数, 在方法的形参前面使用,定义参数名,这个名称可以在mapper中使用
dao接口方法定义
```Java
    Student getStudents(@Param("name")String name,@Param("sex")String sex);
```
mapper文件中写法
```Java
    <select id="getStudents" resultType="com.lqw.mybatis.Bean.Student">
        <!--占位符名要与dao接口中@param注解参数名一致-->
        select from students where name=#{name} and sex=#{sex}
    </select>
```

### 使用对象参数
dao方法的形参是一个 Java对象, 使用这个java对象的属性值作为参数
```Java
    Student getByInfo(Student student);
```

```xml
    <!-- 占位符名要与dao的方法中形参java对象的属性值一致 -->
    <select id="getByInfo"  resultType="com.lqw.mybatis.Bean.Student">
        select from students where name=#{name} and sex=#{sex}
    </select>
```

对象作为dao方法参数时 占位符完整语法格式 : #{property,javaType=Java中数据类型,jdbcType=数据库中类型}
jdbcType:
BIT FLOAT CHAR TIMESTAMP OTHER UNDEFINED
TINYINT REAL VARCHAR BINARY BLOB NVARCHAR
SMALLINT DOUBLE LONGVARCHAR VARBINARY CLOB NCHAR
INTEGER NUMERIC DATE LONGVARBINARY BOOLEAN NCLOB
BIGINT DECIMAL TIME NULL CURSOR ARRAY
    

### 按位置传参(不推荐)
```Java
    List<Student> getStudents1(String name,int age);
```
```xml
    <!--按参数位置传参-->
    <select id="getStudents1"  resultType="com.lqw.mybatis.Bean.Student">
        select from students where name=#{arg0} and sex=#{arg1}
    </select>
```

## 占位符# $
### #占位符
mybatis处理#使用jdbc对象是PrepareStatment对象
```xml
<select id="selectById"par ameterType="integer"
resultType="com.bjpowernode . domain.student">
select id , name , email, age from student where id=#{studentId}</select>
```
```java
//mybatis创建Preparestatement对象，执行sq1语句
string sql=" select id,name , email,age from student where id=?";Preparestatement pst = conn. pr eparestatement(sq7);
pst.setInt(1,1001);//传递参数
Resultset rs= pst.executeQuery();//执行sq1语句
```
#特点:
1) 使用的PrepareStatement对象，执行sql语句，效率高。
2) 使用的PrepareStatement对象，能避免sql语句，sql语句执行更安全。
3) #常常作为***列值***使用的，位于等号的右侧，#0位置的值和数据类型有关的。

### $占位符
mybatis执行${}占位符的sql语句

```xml
<select id="selectById"parameterType="integer "
resultType="com.bjpowernode . domain. student">
select id , name , emai1, age from student where id=${studentId}</select>
```
```Java
//$表示字符串连接，把sq1语句的其他内容和$丹内容使用字符串(+)连接的方式连在一起
string sql="select id ,name , emai1,age from student where id=" +"1001";
//mybatis创建statement对象，执行sq1语句。
statement stmt=conn.createstatement(sq1);
Resultsetrs=stmt.executeQueryO;
```

$的特点
1) 使用Statement对象，执行sql语句，效率低
2) ${}占位符的值，使用的字符串连接方式，有sql注入的风险。有代码安全的问题
3) $数据是原样使用的，不会区分数据类型。
4) $常用作***表名或者列名***，在能保证数据安全的情况下使用$

## 封装输出结果

### ResultSet
(1) 返回自定义对象
```
student selectById(Integer id);
<select id="selectById"par ameterType="integer "
resultType"com.bjpowernode . domain. student">
select id ,name , email,age from student where id=#{studentId}</select>
resultType:现在使用java类型的全限定名称。表示的意思 mybatis执行sq1，把Resultset中的数据转为student类型的对象。 mybatis会做以下操作:
1．调用com.bjpowernode . domain.Student的无参数构造方法，创建对象。
Student student = new Student();//使用反射创建对象
2．同名的列赋值给同名的属性。
student.setId( rs.getInt("id"));
student.setName(rs.getstring ( "name"));
3．得到java对象，如果dao接口返回值是List集合，mybatis把student对象放入到List集合。
所以执行Student mystudent = dao.selectById(1001);得到数据库中id=1001这行数据，
这行数据的列值，付给了mystudent对象的属性。你能得到mystudent对象。就相当于是id=1001这行数据。
```
(2) 返回普通对象

(3) 返回map对象
执行sq1得到一个Map结构数据，mybatis执行sql，把Resultset转为mapsq1执行结果,列名做map的key ,列值作为value
sq1执行得到是一行记录，转为map结构是正确的。
dao接口返回是一个map,sq1语句最多能获取一行记录,多于一行是错误

### ResultMap

resultMap:结果映射。自定义列名和java对象属性的对应关系。常用在列名和属性名不同的情况。用法:
1.先定义resultMap标签，指定列名和属性名称对应关系
2.在select标签使用resultMap属性，指定上面定义的resultMap的id值

```xml
<mapper namespace="com.lqw.mybatis.Dao.PersonDao">

    <!--建立列名和对象属性的一一映射
    column : 列名
    property : 对象属性名
    -->
    <resultMap id="personMap" type="com.lqw.mybatis.Bean.Person">
        <!--主键使用id标签-->
        <id column="pid" property="id"/>
        <!--其他列使用result标签-->
        <result column="pname" property="name"/>
    </resultMap>

    <!--使用resultMap-->
    <select id="getById" resultMap="personMap">
        select pid,pname from person where pid=#{id}
    </select>
```





## 自定义别名
\<typeAliases>标签定义别名,放在\<settings>标签后
```xml
    <typeAliases>
        <!-- 方法一
        type : java对象的全类名
        alias  : 别名
        -->
        <typeAlias type="com.lqw.mybatis.Bean.Student" alias="student"/>
        <!-- 方法二
        name : 包名
        把包中所有类的类名作为别名(不区分大小写)
        -->
        <package name="com.lqw.mybatis.Bean"/>
    </typeAliases>
```

## 解决数据库列名和java对象属性名不一致
1. 使用resultMap(推荐)
2. 在sql语句中使用别名,使列别名与java对象属性名一致



## 模糊查询like
1. 方法一 : 在java程序中把like内容组装好,把这个内容传入sql语句

```xml
       <select id="fuzzyQuery1" resultMap="personMap">
           select * from person where pname like #{pname}
       </select>
```

```Java
       @Test
       public void testfuzzyQuery(){
           SqlSession session= MybatisTool.getSqlSession();
           PersonDao dao=session.getMapper(PersonDao.class);
           //在Java程序中组装
           String sqlname="%lib%";
           List<Person> person=dao.fuzzyQuery1(sqlname);
           System.out.println(person);
           session.close();
       }
```

   

2. 方法二 :  在sql语句中组装like语句

```xml
       <select id="fuzzyQuery2" resultMap="personMap">
        select * from person where pname like "%" #{pname} "%"
    </select>
```

```Java
       @Test
    public void testfuzzyQuery1(){
        SqlSession session= MybatisTool.getSqlSession();
        PersonDao dao=session.getMapper(PersonDao.class);
        String sqlname="lib";
        List<Person> person=dao.fuzzyQuery2(sqlname);
        System.out.println(person);
        session.close();
    }
```

# 动态SQL

多条件查询时, 同一个dao方法, 根据不同的条件执行不同的sql语句, 只要是where部分有变化
使用mybatis提供的标签, 实现动态sql能力:
## 实体符号
在mapper的动态sql中若出现大于号(>)/小于号(<), 需要将其转换成实体符号, 否则xml会出现解析出错
|符号 |符号实体|
|----|----|
| < | \&lt; |
| > | \&gt; |

## if标签
```xml
<if test="判断语句 boolean类型 使用java语法">
    test==true时 执行的sql语句
</if>
```
## where标签
使用if标签时，容易引起sql语句语法错误。使用where标签解决if产生的语法问题。
使用时where ,里面是一个或多个if标签，当有一个f标签判断条件为true, where标签会转为WHERE关键字附加到sql语句的后面。如果if没有一个条件为true，忽略where和里面的if。I
where标签删除和他最近的or或者and。
```xml
    <select id="dynamicSql" resultMap="personMap">
        select * from person
        <where>
            <if test="name!=null and name!=''"> or pname=#{name}</if>
            <if test="id!=null and id!=''">or pid=#{id}</if>
        </where>
    </select>
```

## foreach标签
使用foreach可以循环数组，list集合，一般使用在in语句中。
语法:
```xml
\<foreach collection="集合类型" open="开始的字符" close="结束的字符” 
item="集合中的成员" separator="集合成员之间的分隔符">
#{item的值]
\</ foreach>
```
标签属性:
collection : 表示，循环的对象是数组，还是1ist集合。如果dao接口方法的形参是数组，
collection="array”,如果dao接口形参是List，collection="list"
open:循环开始时的字符。sql.append(""(");
close:循环结束时字符。sql.append("")");
item:集合成员，自定义的变量。 Integer item = idlist.get(i);// item是集合成员
separator:集合成员之间的分隔符。sq1 .append(",");//集合成员之间的分隔符

### 遍历List<简单类型>
```xml
    <select id="testForeach" resultMap="personMap">
        select * from person where pid in
        <foreach collection="list" open="(" close=")" item="id" separator=",">
            #{id}
        </foreach>

    </select>
```

### 遍历List<对象类型>
```xml
    <select id="testForeach1" resultMap="personMap">
        select * from person
        <if test="list!=null and list.size>0">
            where pid in
            <foreach collection="list" open="(" close=")" item="p" separator=",">
                <!--这里使用 . 运算符获取属性值-->
                #{p.id}
            </foreach>
        </if>
    </select>
```

## sql标签
sql标签标示一段sql代码，可以是表名，几个字段, where条件都可以，可以在其他地方复用sql标签的内容。使用方式:
在mapper文件中定义sq1代码片段
   `<sq1 id="sql标签唯一标识">部分sq1语句</sql>2>`
在其他的位置，使用include标签引用某个代码片段
   `<include refid="sql id" />`

# Mybatis配置文件

## 主配置文件

Mybatis配置项的顺序不能颠倒，如果颠倒了它们的顺序，那么mybatis启动阶段会发生异常，导致程序无法运行.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration><!--配置-->
	<properties/><!--属性-->
	<settings/><!--设置-->
	<typeAliases/><!--类型别名--> 
	<typeHandlers/><!--类型处理器--> 
	<objectFactory/><!--对象工厂-->  
	<plugins/><!--插件--> 
	<environments><!--配置环境--> 
		<environment><!--环境变量--> 
		<transactionManager/><!--事务管理器--> 
			<dataSource/><!--数据源--> 
		</environment>
	</environments>
	<databaseidProvider/><!--数据库厂商标识-->  
	<mappers/><!--映射器--> 
</configuration>
```

### \<settings>
mybatis的全局设置, 影响整个mybatis的运行, 一般保持默认设置就行, 需要修改时查询文档即可.

### \<environments>
environments:环境标签，在他里面可以配置多个environment
   属性: default 必须是某个环境的id值,标识mybatis默认链接的数据库
environment:表示一个数据库的连接信息。
   属性:id 自定义的环境的标识。唯一值。
transactionManager :事务管理器
   属性:type 表示事务管理器的类型。
   属性值:
   1)JDBC:使用Connection对象，由mybatis自己完成事务的处理oT
   2)MANAGED:管理，表示把事务的处理交给容器实现（由其他软件完成事务的提交，回滚)
dataSource:数据源，创建的connection对象，连接数据库。
   属性: type 数据源的类型
   属性值：
   1)POOLED，会在内存中创建PooledDataSource类，管理多个Connection连接对象，使用的连接池
   2)UNPOOLED ，不使用连接池，mybatis创建一个UnPooledDataSource这个类，每次执行sq1
语句先创建Connection对象，再执行sq1语句，最后关闭Connection
   3)JNDI : java的命名和目录服务。

### \<mappers>
引入mapper文件的两种方式
1. \<mapper> 引入单个mapper文件
`<mapper resource="com/lqw/mybatis/Dao/PerDao.xml"/>`

2. \<package> 引入包下所有mapper文件 
`<package name="com.lqw.mybatis.Dao"/>`

## 使用数据库属性配置文件
需要把数据库的配置信息放到一个单独文件中，独立管理。这个文件扩展名是properties.在这个文件中，使用自定义的key=value的格式表示数据

使用步骤:
1. 在resources目录中，创建xxxx.properties
   在主配置文件中使用properties标签引入外部配置文件`<properties resource="jdbc.properties"/>`
2. 在文件中，使用key=value的格式定义数据。
   例如 jdbc.url=jdbc:mysq://localhost:3306/springdb
3. 在mybatis主配置文件，使用property标签引用外部的属性配置文件
4. 在使用值的位置，使用${key}获取key对应的value(等号右侧的值)

# 扩展PageHelper

1. maven坐标

```xml
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>5.1.10</version>
        </dependency>
```
2. plugin配置

```xml
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
    </plugins>
```
3. PageHelper对象

查询语句之前调用PageHelper.startPage静态方法。
除了PageHelper.startPage方法外，还提供了类似用法的 PageHelper.offsetPage方法。在你需要进行分页的 MyBatis查询方法前调用PageHelper.startPage静态方法即可，紧跟在这个方法后的第一个MyBatis查询方法会被进行分页。这里的实质是插件代写sql 中的limit语句.
```Java
   //int pageNum : 第几页
   //int pageSize : 每页条数
    public static <E> Page<E> startPage(int pageNum, int pageSize)
```


# 逆向工程 MyBatis Generator
## 需要的文件
1. 数据库驱动
2. 配置文件 generatorConfig.xml
3. 官方jar包
maven配置方法
```xml

<!-- mybatis代码生成插件 -->
      <plugin>
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-maven-plugin</artifactId>
        <version>1.3.6</version>
        <configuration>
          <!--配置文件位置-->
          <configurationFile>src/main/webapp/WEB-INF/generatorConfig.xml</configurationFile>
          <verbose>true</verbose>
          <overwrite>true</overwrite>
        </configuration>
        <dependencies>
          <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.6</version>
          </dependency>
          <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.26</version>
          </dependency>
        </dependencies>
      </plugin>
```

## 运行流程
1. 链接数据库
2. 从数据库中获取字段
3. 依赖字段生成mapper文件/dao/实体类

## generatorConfig.xml配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<!-- 配置生成器 -->
<generatorConfiguration>
<!-- 可以用于加载配置项或者配置文件，在整个配置文件中就可以使用${propertyKey}的方式来引用配置项
    resource：配置资源加载地址，使用resource，MBG从classpath开始找，比如com/myproject/generatorConfig.properties        
    url：配置资源加载地质，使用URL的方式，比如file:///C:/myfolder/generatorConfig.properties.
    注意，两个属性只能选址一个;

    另外，如果使用了mybatis-generator-maven-plugin，那么在pom.xml中定义的properties都可以直接在generatorConfig.xml中使用
<properties resource="" url="" />
 -->

 <!-- 在MBG工作的时候，需要额外加载的依赖包
     location属性指明加载jar/zip包的全路径
<classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />
  -->

<!-- 
    context:生成一组对象的环境 
    id:必选，上下文id，用于在生成错误时提示
    defaultModelType:指定生成对象的样式
        1，conditional：类似hierarchical；
        2，flat：所有内容（主键，blob）等全部生成在一个对象中；
        3，hierarchical：主键生成一个XXKey对象(key class)，Blob等单独生成一个对象，其他简单属性在一个对象中(record class)
    targetRuntime:
        1，MyBatis3：默认的值，生成基于MyBatis3.x以上版本的内容，包括XXXBySample；
        2，MyBatis3Simple：类似MyBatis3，只是不生成XXXBySample；
    introspectedColumnImpl：类全限定名，用于扩展MBG
-->
<context id="mysql" defaultModelType="hierarchical" targetRuntime="MyBatis3Simple" >

    <!-- 自动识别数据库关键字，默认false，如果设置为true，根据SqlReservedWords中定义的关键字列表；
        一般保留默认值，遇到数据库关键字（Java关键字），使用columnOverride覆盖
     -->
    <property name="autoDelimitKeywords" value="false"/>
    <!-- 生成的Java文件的编码 -->
    <property name="javaFileEncoding" value="UTF-8"/>
    <!-- 格式化java代码 -->
    <property name="javaFormatter" value="org.mybatis.generator.api.dom.DefaultJavaFormatter"/>
    <!-- 格式化XML代码 -->
    <property name="xmlFormatter" value="org.mybatis.generator.api.dom.DefaultXmlFormatter"/>

    <!-- beginningDelimiter和endingDelimiter：指明数据库的用于标记数据库对象名的符号，比如ORACLE就是双引号，MYSQL默认是`反引号； -->
    <property name="beginningDelimiter" value="`"/>
    <property name="endingDelimiter" value="`"/>

    <!-- 必须要有的，使用这个配置链接数据库
        @TODO:是否可以扩展
     -->
    <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql:///pss" userId="root" password="admin">
        <!-- 这里面可以设置property属性，每一个property属性都设置到配置的Driver上 -->
    </jdbcConnection>

    <!-- java类型处理器 
        用于处理DB中的类型到Java中的类型，默认使用JavaTypeResolverDefaultImpl；
        注意一点，默认会先尝试使用Integer，Long，Short等来对应DECIMAL和 NUMERIC数据类型； 
    -->
    <javaTypeResolver type="org.mybatis.generator.internal.types.JavaTypeResolverDefaultImpl">
        <!-- 
            true：使用BigDecimal对应DECIMAL和 NUMERIC数据类型
            false：默认,
                scale>0;length>18：使用BigDecimal;
                scale=0;length[10,18]：使用Long；
                scale=0;length[5,9]：使用Integer；
                scale=0;length<5：使用Short；
         -->
        <property name="forceBigDecimals" value="false"/>
    </javaTypeResolver>


    <!-- java模型创建器，是必须要的元素
        负责：1，key类（见context的defaultModelType）；2，java类；3，查询类
        targetPackage：生成的类要放的包，真实的包受enableSubPackages属性控制；
        targetProject：目标项目，指定一个存在的目录下，生成的内容会放到指定目录中，如果目录不存在，MBG不会自动建目录
     -->
    <javaModelGenerator targetPackage="com._520it.mybatis.domain" targetProject="src/main/java">
        <!--  for MyBatis3/MyBatis3Simple
            自动为每一个生成的类创建一个构造方法，构造方法包含了所有的field；而不是使用setter；
         -->
        <property name="constructorBased" value="false"/>

        <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
        <property name="enableSubPackages" value="true"/>

        <!-- for MyBatis3 / MyBatis3Simple
            是否创建一个不可变的类，如果为true，
            那么MBG会创建一个没有setter方法的类，取而代之的是类似constructorBased的类
         -->
        <property name="immutable" value="false"/>

        <!-- 设置一个根对象，
            如果设置了这个根对象，那么生成的keyClass或者recordClass会继承这个类；在Table的rootClass属性中可以覆盖该选项
            注意：如果在key class或者record class中有root class相同的属性，MBG就不会重新生成这些属性了，包括：
                1，属性名相同，类型相同，有相同的getter/setter方法；
         -->
        <property name="rootClass" value="com._520it.mybatis.domain.BaseDomain"/>

        <!-- 设置是否在getter方法中，对String类型字段调用trim()方法 -->
        <property name="trimStrings" value="true"/>
    </javaModelGenerator>


    <!-- 生成SQL map的XML文件生成器，
        注意，在Mybatis3之后，我们可以使用mapper.xml文件+Mapper接口（或者不用mapper接口），
            或者只使用Mapper接口+Annotation，所以，如果 javaClientGenerator配置中配置了需要生成XML的话，这个元素就必须配置
        targetPackage/targetProject:同javaModelGenerator
     -->
    <sqlMapGenerator targetPackage="com._520it.mybatis.mapper" targetProject="src/main/resources">
        <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
        <property name="enableSubPackages" value="true"/>
    </sqlMapGenerator>


    <!-- 对于mybatis来说，即生成Mapper接口，注意，如果没有配置该元素，那么默认不会生成Mapper接口 
        targetPackage/targetProject:同javaModelGenerator
        type：选择怎么生成mapper接口（在MyBatis3/MyBatis3Simple下）：
            1，ANNOTATEDMAPPER：会生成使用Mapper接口+Annotation的方式创建（SQL生成在annotation中），不会生成对应的XML；
            2，MIXEDMAPPER：使用混合配置，会生成Mapper接口，并适当添加合适的Annotation，但是XML会生成在XML中；
            3，XMLMAPPER：会生成Mapper接口，接口完全依赖XML；
        注意，如果context是MyBatis3Simple：只支持ANNOTATEDMAPPER和XMLMAPPER
    -->
    <javaClientGenerator targetPackage="com._520it.mybatis.mapper" type="ANNOTATEDMAPPER" targetProject="src/main/java">
        <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
        <property name="enableSubPackages" value="true"/>

        <!-- 可以为所有生成的接口添加一个父接口，但是MBG只负责生成，不负责检查
        <property name="rootInterface" value=""/>
         -->
    </javaClientGenerator>

    <!-- 选择一个table来生成相关文件，可以有一个或多个table，必须要有table元素
        选择的table会生成一下文件：
        1，SQL map文件
        2，生成一个主键类；
        3，除了BLOB和主键的其他字段的类；
        4，包含BLOB的类；
        5，一个用户生成动态查询的条件类（selectByExample, deleteByExample），可选；
        6，Mapper接口（可选）

        tableName（必要）：要生成对象的表名；
        注意：大小写敏感问题。正常情况下，MBG会自动的去识别数据库标识符的大小写敏感度，在一般情况下，MBG会
            根据设置的schema，catalog或tablename去查询数据表，按照下面的流程：
            1，如果schema，catalog或tablename中有空格，那么设置的是什么格式，就精确的使用指定的大小写格式去查询；
            2，否则，如果数据库的标识符使用大写的，那么MBG自动把表名变成大写再查找；
            3，否则，如果数据库的标识符使用小写的，那么MBG自动把表名变成小写再查找；
            4，否则，使用指定的大小写格式查询；
        另外的，如果在创建表的时候，使用的""把数据库对象规定大小写，就算数据库标识符是使用的大写，在这种情况下也会使用给定的大小写来创建表名；
        这个时候，请设置delimitIdentifiers="true"即可保留大小写格式；

        可选：
        1，schema：数据库的schema；
        2，catalog：数据库的catalog；
        3，alias：为数据表设置的别名，如果设置了alias，那么生成的所有的SELECT SQL语句中，列名会变成：alias_actualColumnName
        4，domainObjectName：生成的domain类的名字，如果不设置，直接使用表名作为domain类的名字；可以设置为somepck.domainName，那么会自动把domainName类再放到somepck包里面；
        5，enableInsert（默认true）：指定是否生成insert语句；
        6，enableSelectByPrimaryKey（默认true）：指定是否生成按照主键查询对象的语句（就是getById或get）；
        7，enableSelectByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询语句；
        8，enableUpdateByPrimaryKey（默认true）：指定是否生成按照主键修改对象的语句（即update)；
        9，enableDeleteByPrimaryKey（默认true）：指定是否生成按照主键删除对象的语句（即delete）；
        10，enableDeleteByExample（默认true）：MyBatis3Simple为false，指定是否生成动态删除语句；
        11，enableCountByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询总条数语句（用于分页的总条数查询）；
        12，enableUpdateByExample（默认true）：MyBatis3Simple为false，指定是否生成动态修改语句（只修改对象中不为空的属性）；
        13，modelType：参考context元素的defaultModelType，相当于覆盖；
        14，delimitIdentifiers：参考tableName的解释，注意，默认的delimitIdentifiers是双引号，如果类似MYSQL这样的数据库，使用的是`（反引号，那么还需要设置context的beginningDelimiter和endingDelimiter属性）
        15，delimitAllColumns：设置是否所有生成的SQL中的列名都使用标识符引起来。默认为false，delimitIdentifiers参考context的属性

        注意，table里面很多参数都是对javaModelGenerator，context等元素的默认属性的一个复写；
     -->
    <table tableName="userinfo" >

        <!-- 参考 javaModelGenerator 的 constructorBased属性-->
        <property name="constructorBased" value="false"/>

        <!-- 默认为false，如果设置为true，在生成的SQL中，table名字不会加上catalog或schema； -->
        <property name="ignoreQualifiersAtRuntime" value="false"/>

        <!-- 参考 javaModelGenerator 的 immutable 属性 -->
        <property name="immutable" value="false"/>

        <!-- 指定是否只生成domain类，如果设置为true，只生成domain类，如果还配置了sqlMapGenerator，那么在mapper XML文件中，只生成resultMap元素 -->
        <property name="modelOnly" value="false"/>

        <!-- 参考 javaModelGenerator 的 rootClass 属性 
        <property name="rootClass" value=""/>
         -->

        <!-- 参考javaClientGenerator 的  rootInterface 属性
        <property name="rootInterface" value=""/>
        -->

        <!-- 如果设置了runtimeCatalog，那么在生成的SQL中，使用该指定的catalog，而不是table元素上的catalog 
        <property name="runtimeCatalog" value=""/>
        -->

        <!-- 如果设置了runtimeSchema，那么在生成的SQL中，使用该指定的schema，而不是table元素上的schema 
        <property name="runtimeSchema" value=""/>
        -->

        <!-- 如果设置了runtimeTableName，那么在生成的SQL中，使用该指定的tablename，而不是table元素上的tablename 
        <property name="runtimeTableName" value=""/>
        -->

        <!-- 注意，该属性只针对MyBatis3Simple有用；
            如果选择的runtime是MyBatis3Simple，那么会生成一个SelectAll方法，如果指定了selectAllOrderByClause，那么会在该SQL中添加指定的这个order条件；
         -->
        <property name="selectAllOrderByClause" value="age desc,username asc"/>

        <!-- 如果设置为true，生成的model类会直接使用column本身的名字，而不会再使用驼峰命名方法，比如BORN_DATE，生成的属性名字就是BORN_DATE,而不会是bornDate -->
        <property name="useActualColumnNames" value="false"/>


        <!-- generatedKey用于生成生成主键的方法，
            如果设置了该元素，MBG会在生成的<insert>元素中生成一条正确的<selectKey>元素，该元素可选
            column:主键的列名；
            sqlStatement：要生成的selectKey语句，有以下可选项：
                Cloudscape:相当于selectKey的SQL为： VALUES IDENTITY_VAL_LOCAL()
                DB2       :相当于selectKey的SQL为： VALUES IDENTITY_VAL_LOCAL()
                DB2_MF    :相当于selectKey的SQL为：SELECT IDENTITY_VAL_LOCAL() FROM SYSIBM.SYSDUMMY1
                Derby      :相当于selectKey的SQL为：VALUES IDENTITY_VAL_LOCAL()
                HSQLDB      :相当于selectKey的SQL为：CALL IDENTITY()
                Informix  :相当于selectKey的SQL为：select dbinfo('sqlca.sqlerrd1') from systables where tabid=1
                MySql      :相当于selectKey的SQL为：SELECT LAST_INSERT_ID()
                SqlServer :相当于selectKey的SQL为：SELECT SCOPE_IDENTITY()
                SYBASE      :相当于selectKey的SQL为：SELECT @@IDENTITY
                JDBC      :相当于在生成的insert元素上添加useGeneratedKeys="true"和keyProperty属性
        <generatedKey column="" sqlStatement=""/>
         -->

        <!-- 
            该元素会在根据表中列名计算对象属性名之前先重命名列名，非常适合用于表中的列都有公用的前缀字符串的时候，
            比如列名为：CUST_ID,CUST_NAME,CUST_EMAIL,CUST_ADDRESS等；
            那么就可以设置searchString为"^CUST_"，并使用空白替换，那么生成的Customer对象中的属性名称就不是
            custId,custName等，而是先被替换为ID,NAME,EMAIL,然后变成属性：id，name，email；

            注意，MBG是使用java.util.regex.Matcher.replaceAll来替换searchString和replaceString的，
            如果使用了columnOverride元素，该属性无效；

        <columnRenamingRule searchString="" replaceString=""/>
         -->


         <!-- 用来修改表中某个列的属性，MBG会使用修改后的列来生成domain的属性；
             column:要重新设置的列名；
             注意，一个table元素中可以有多个columnOverride元素哈~
          -->
         <columnOverride column="username">
             <!-- 使用property属性来指定列要生成的属性名称 -->
             <property name="property" value="userName"/>

             <!-- javaType用于指定生成的domain的属性类型，使用类型的全限定名
             <property name="javaType" value=""/>
              -->

             <!-- jdbcType用于指定该列的JDBC类型 
             <property name="jdbcType" value=""/>
              -->

             <!-- typeHandler 用于指定该列使用到的TypeHandler，如果要指定，配置类型处理器的全限定名
                 注意，mybatis中，不会生成到mybatis-config.xml中的typeHandler
                 只会生成类似：where id = #{id,jdbcType=BIGINT,typeHandler=com._520it.mybatis.MyTypeHandler}的参数描述
             <property name="jdbcType" value=""/>
             -->

             <!-- 参考table元素的delimitAllColumns配置，默认为false
             <property name="delimitedColumnName" value=""/>
              -->
         </columnOverride>

         <!-- ignoreColumn设置一个MGB忽略的列，如果设置了改列，那么在生成的domain中，生成的SQL中，都不会有该列出现 
             column:指定要忽略的列的名字；
             delimitedColumnName：参考table元素的delimitAllColumns配置，默认为false

             注意，一个table元素中可以有多个ignoreColumn元素
         <ignoreColumn column="deptId" delimitedColumnName=""/>
         -->
    </table>

</context>

</generatorConfiguration>
```

###  配置文件头

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
```

### 根节点

```xml
<generatorConfiguration>
    <!-- 具体配置内容 -->
</generatorConfiguration>  
```
根节点子元素,配置以下元素有严格顺序

#### 1. \<properties> (0个或1个)

这个元素用来指定外部的属性元素，不是必须的元素。

元素用于指定一个需要在配置中解析使用的外部属性文件，引入属性文件后，可以在配置中使用 `${property}`这种形式的引用，通过这种方式引用属性文件中的属性值。 对于后面需要配置的**jdbc信息**和`targetProject`属性会很有用。

这个属性可以通过`resource`或者`url`来指定属性文件的位置，这两个属性只能使用其中一个来指定，同时出现会报错。

- `resource`：指定**classpath**下的属性文件，使用类似`com/myproject/generatorConfig.properties`这样的属性值。
- `url`：可以指定文件系统上的特定位置，例如`file:///C:/myfolder/generatorConfig.properties`

#### 2.  \<classPathEntry> (0个或多个)

最常见的用法是通过这个属性指定驱动的路径，例如：

```
<classPathEntry location="E:\mysql\mysql-connector-java-5.1.29.jar"/>
```

**注意，classPathEntry只在下面这两种情况下才有效**：

- 当加载 JDBC 驱动内省数据库时
- 当加载根类中的 JavaModelGenerator 检查重写的方法时

**因此，**如果你需要加载其他用途的jar包，**classPathEntry起不到作用**，不能这么写，解决的办法就是将你用的jar包添加到类路径中.

#### 3. \<context> (1个或多个)

`<context>`元素用于指定生成一组对象的环境。例如指定要连接的数据库，要生成对象的类型和要处理的数据库中的表。运行MBG的时候还可以指定要运行的`<context>`。

该元素只有一个**必选属性**`id`，用来唯一确定一个`<context>`元素，该`id`属性可以在运行MBG的使用。

此外还有几个**可选属性**：

- `defaultModelType`:**这个属性很重要**，这个属性定义了MBG如何生成**实体类**。
  这个属性有以下可选值：
  - `conditional`:*这是默认值*,这个模型和下面的`hierarchical`类似，除了如果那个单独的类将只包含一个字段，将不会生成一个单独的类。 因此,如果一个表的主键只有一个字段,那么不会为该字段生成单独的实体类,会将该字段合并到基本实体类中。
  - `flat`:该模型为每一张表只生成一个实体类。这个实体类包含表中的所有字段。**这种模型最简单，推荐使用。**
  - `hierarchical`:如果表有主键,那么该模型会产生一个单独的主键实体类,如果表还有BLOB字段， 则会为表生成一个包含所有BLOB字段的单独的实体类,然后为所有其他的字段生成一个单独的实体类。 MBG会在所有生成的实体类之间维护一个继承关系。
- `targetRuntime`:此属性用于指定生成的代码的运行时环境。该属性支持以下可选值：
  - `MyBatis3`:*这是默认值*
  - `MyBatis3Simple`
  - `Ibatis2Java2`
  - `Ibatis2Java5` 一般情况下使用默认值即可，有关这些值的具体作用以及区别请查看中文文档的详细内容。
- `introspectedColumnImpl`:该参数可以指定扩展`org.mybatis.generator.api.IntrospectedColumn`该类的实现类。该属性的作用可以查看[扩展MyBatis Generator](http://mbg.cndocs.tk/reference/extending.html)。

一般情况下，我们使用如下的配置即可：

```
<context id="Mysql" defaultModelType="flat">
```

如果你希望不生成和`Example`查询有关的内容，那么可以按照如下进行配置:

```
<context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
```

使用`MyBatis3Simple`可以避免在后面的`<table>`中逐个进行配置。

MBG配置中的其他几个元素，基本上都是`<context>`的子元素，这些子元素（有严格的配置顺序）包括：

* \<property> (0个或多个)
* \<plugin> (0个或多个)
* \<commentGenerator>   (0个或多个)
* \<jdbcConnection> (1个)
* \<javaTypeResolver> (0个或多个)
* \<javaModelGenerator>(1个)
* \<sqlMapGenerator> (0个或多个)
* \<javaClientGenerator> (0个或多个)
* \<table> (0个或多个)

### \<context>元素

#### 1 . \<property> (0个或多个)

属性:

- `autoDelimitKeywords`
- `beginningDelimiter`
- `endingDelimiter`
- `javaFileEncoding`
- `javaFormatter`
- `xmlFormatter`

`autoDelimitKeywords`，当表名或者字段名为SQL关键字的时候，可以设置该属性为true，MBG会自动给表名或字段名添加**分隔符**。
由于`beginningDelimiter`和`endingDelimiter`的默认值为双引号(`"`)，在Mysql中不能这么写，所以还要将这两个默认值改为**反单引号(`)**，配置如下：

```
<property name="beginningDelimiter" value="`"/>
<property name="endingDelimiter" value="`"/>  
```

属性`javaFileEncoding`设置要使用的Java文件的编码，默认使用当前平台的编码，只有当生产的编码需要特殊指定时才需要使用，一般用不到。

最后两个`javaFormatter`和`xmlFormatter`属性**可能会**很有用，如果你想使用模板来定制生成的java文件和xml文件的样式，你可以通过指定这两个属性的值来实现。

接下来分节对其他的子元素逐个进行介绍。

#### 2 . \<plugin> (0个或多个)



#### 3 . \<commentGenerator>   (0个或多个)
#### 4 . \<jdbcConnection> (1个)

用于指定数据库连接信息，该元素必选，并且只能有一个。

配置该元素只需要注意如果JDBC驱动不在**classpath**下，就需要通过`<classPathEntry>`元素引入jar包，这里**推荐**将jar包放到**classpath**下。

#### 5 . \<javaTypeResolver> (0个或多个)
#### 6 . \<javaModelGenerator>(1个)

该元素用来控制生成的实体类，根据`<context>`中配置的`defaultModelType`，一个表可能会对应生成多个不同的实体类。一个表对应多个类实际上并不方便，所以前面也推荐使用`flat`，这种情况下一个表对应一个实体类。

该元素只有两个属性，都是必选的。

- `targetPackage`:生成实体类存放的包名，一般就是放在该包下。实际还会受到其他配置的影响(`<table>`中会提到)。
- `targetProject`:指定目标项目路径，使用的是文件系统的绝对路径。

该元素支持以下几个`<property>`子元素属性：

- `constructorBased`:该属性只对`MyBatis3`有效，如果`true`就会使用构造方法入参，如果`false`就会使用`setter`方式。默认为`false`。
- `enableSubPackages`:如果`true`，MBG会根据`catalog`和`schema`来生成子包。如果`false`就会直接用`targetPackage`属性。默认为`false`。
- `immutable`:该属性用来配置实体类属性是否可变，如果设置为`true`，那么`constructorBased`不管设置成什么，都会使用构造方法入参，并且不会生成`setter`方法。如果为`false`，实体类属性就可以改变。默认为`false`。
- `rootClass`:设置所有实体类的基类。如果设置，需要使用类的全限定名称。并且如果MBG能够加载`rootClass`，那么MBG不会覆盖和父类中完全匹配的属性。匹配规则：
  - 属性名完全相同
  - 属性类型相同
  - 属性有`getter`方法
  - 属性有`setter`方法
- `trimStrings`:是否对数据库查询结果进行`trim`操作，如果设置为`true`就会生成类似这样`public void setUsername(String username) {this.username = username == null ? null : username.trim();}`的`setter`方法。默认值为`false`。

#### 7 . \<sqlMapGenerator> (0个或多个)

如果targetRuntime目标是**iBATIS2**，该元素必须配置一个。
如果targetRuntime目标是**MyBatis3**，只有当\<javaClientGenerator>需要XML时，该元素必须配置一个。 如果没有配置\<javaClientGenerator>，则使用以下的规则：
    如果指定了一个\<sqlMapGenerator>，那么MBG将只生成XML的SQL映射文件和实体类。
    如果没有指定\<sqlMapGenerator>，那么MBG将只生成实体类。

该元素只有两个属性（和前面提过的\<javaModelGenerator>的属性含义一样），都是必选的。
targetPackage:生成实体类存放的包名，一般就是放在该包下。
targetProject:指定目标项目路径，使用的是文件系统的绝对路径。
该元素支持\<property>子元素，只有一个可以配置的属性：
    enableSubPackages: 如果true，MBG会根据catalog和schema来生成子包。如果false就会直接用targetPackage属性。默认为false。

#### 8 . \<javaClientGenerator> (0个或多个)

#### 9.  \<table> (1个或多个)

必选属性:

- `tableName`：指定要生成的表名，可以使用[SQL通配符](http://www.w3school.com.cn/sql/sql_wildcards.asp)匹配多个表。

可选属性:

- `schema`:数据库的schema,可以使用[SQL通配符](http://www.w3school.com.cn/sql/sql_wildcards.asp)匹配。如果设置了该值，生成SQL的表名会变成如`schema.tableName`的形式。
- `catalog`:数据库的catalog，如果设置了该值，生成SQL的表名会变成如`catalog.tableName`的形式。
- `alias`:如果指定，这个值会用在生成的select查询SQL的表的别名和列名上。 列名会被别名为 alias_actualColumnName(别名_实际列名) 这种模式。
- `domainObjectName`:生成对象的基本名称。如果没有指定，MBG会自动根据表名来生成名称。
- `enableXXX`:XXX代表多种SQL方法，该属性用来指定是否生成对应的XXX语句。
- `selectByPrimaryKeyQueryId`:DBA跟踪工具会用到，具体请看详细文档。
- `selectByExampleQueryId`:DBA跟踪工具会用到，具体请看详细文档。
- `modelType`:和`<context>`的`defaultModelType`含义一样，这里可以针对表进行配置，这里的配置会覆盖`<context>`的`defaultModelType`配置。
- `escapeWildcards`:这个属性表示当查询列，是否对schema和表名中的SQL通配符 ('_' and '%') 进行转义。 对于某些驱动当schema或表名中包含SQL通配符时（例如，一个表名是MY_TABLE，有一些驱动需要将下划线进行转义）是必须的。默认值是`false`。
- `delimitIdentifiers`:是否给标识符增加**分隔符**。默认`false`。当`catalog`,`schema`或`tableName`中包含空白时，默认为`true`。
- `delimitAllColumns`:是否对所有列添加**分隔符**。默认`false`。

该元素包含多个可用的`<property>`子元素，可选属性为：

- `constructorBased`:和`<javaModelGenerator>`中的属性含义一样。
- `ignoreQualifiersAtRuntime`:生成的SQL中的表名将不会包含`schema`和`catalog`前缀。
- `immutable`:和`<javaModelGenerator>`中的属性含义一样。
- `modelOnly`:此属性用于配置是否为表只生成实体类。如果设置为`true`就不会有Mapper接口。如果配置了`<sqlMapGenerator>`，并且`modelOnly`为`true`，那么XML映射文件中只有实体对象的映射元素(`<resultMap>`)。如果为`true`还会覆盖属性中的`enableXXX`方法，将不会生成任何CRUD方法。
- `rootClass`:和`<javaModelGenerator>`中的属性含义一样。
- `rootInterface`:和`<javaClientGenerator>`中的属性含义一样。
- `runtimeCatalog`:运行时的`catalog`，当生成表和运行环境的表的`catalog`不一样的时候可以使用该属性进行配置。
- `runtimeSchema`:运行时的`schema`，当生成表和运行环境的表的`schema`不一样的时候可以使用该属性进行配置。
- `runtimeTableName`:运行时的`tableName`，当生成表和运行环境的表的`tableName`不一样的时候可以使用该属性进行配置。
- `selectAllOrderByClause`:该属性值会追加到`selectAll`方法后的SQL中，会直接跟`order by`拼接后添加到SQL末尾。
- `useActualColumnNames`:如果设置为true,那么MBG会使用从数据库元数据获取的列名作为生成的实体对象的属性。 如果为false(默认值)，MGB将会尝试将返回的名称转换为驼峰形式。 在这两种情况下，可以通过 元素显示指定，在这种情况下将会忽略这个（useActualColumnNames）属性。
- `useColumnIndexes`:如果是true,MBG生成resultMaps的时候会使用列的索引,而不是结果中列名的顺序。
- `useCompoundPropertyNames`:如果是true,那么MBG生成属性名的时候会将列名和列备注接起来. 这对于那些通过第四代语言自动生成列(例如:FLD22237),但是备注包含有用信息(例如:"customer id")的数据库来说很有用. 在这种情况下,MBG会生成属性名FLD2237_CustomerId。

除了`<property>`子元素外，`<table>`还包含以下子元素：

