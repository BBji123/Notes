

## 快速开始
基于springboot

1. 引入依赖

```xml
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.4.3.4</version>
    </dependency>
```

2. 配置
```yaml
# DataSource Config
```
 在springboot启动类添加@MapperScan注解

3. 编码
实体类User
```java

```

UserMapper类(Dao)
```java
public interface UserMapper extends BaseMapper<User> {

}
```

测试
```java
@SpringBootTest(classes = MyApplication.class)
public class Test01 {
    @Autowired
    public UserMapper userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userMapper.selectList(null);
        //Assert.assertEquals(5, userList.size());
        userList.forEach(System.out::println);
    }
}
```

## 配置日志
```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## CURD拓展

### 主键生成策略
1. 雪花算法
2. 主键自增
3. UUID


### 自动填充
一般数据表中都要有create_tim和update_time,这两个字段需要自动填充
1. 数据库级别
不建议,数据库设计完成后一般不允许修改
在数据库中设置两个字段的自动填充

2. 代码级别  
    1. 注解填充字段
```java
public class User {

    // 注意！这里需要标记为填充字段
    @TableField(.. fill = FieldFill.INSERT)
    private String fillField;

    ....
}
FieldFill
值	描述
DEFAULT	默认不处理
INSERT	插入时填充字段
UPDATE	更新时填充字段
INSERT_UPDATE	插入和更新时填充字段


```
    2. 自定义实现类
```java
@Slf4j
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("start insert fill ....");
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now()); // 起始版本 3.3.0(推荐使用)
        // 或者
       this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now()); // 起始版本 3.3.3(推荐)
        // 或者
        this.fillStrategy(metaObject, "createTime", LocalDateTime.now()); // 也可以使用(3.3.0 该方法有bug)
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("start update fill ....");
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now()); // 起始版本 3.3.0(推荐)
        // 或者
        this.strictUpdateFill(metaObject, "updateTime", () -> LocalDateTime.now(), LocalDateTime.class); // 起始版本 3.3.3(推荐)
        // 或者
        this.fillStrategy(metaObject, "updateTime", LocalDateTime.now()); // 也可以使用(3.3.0 该方法有bug)
    }
}
```

### 乐观锁
1. 添加乐观锁字段标记
```java
@Version
private int version;
```
>说明:
支持的数据类型只有:int,Integer,long,Long,Date,Timestamp,LocalDateTime
整数类型下 newVersion = oldVersion + 1
newVersion 会回写到 entity 中
仅支持 updateById(id) 与 update(entity, wrapper) 方法
在 update(entity, wrapper) 方法下, wrapper 不能复用!!!


2. 注册乐观锁插件
```java
@Configuration
//Mapper文件自动扫描
@MapperScan("com.lqw.boot.mapper")
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
}

```
3. 测试
```java
    @Test
    public void testSelect() {
        User user=userMapper.selectById(7L);
        user.setName("liqiwei");
        userMapper.updateById(user);

    }
```

### 分页查询

1.  注册分页插件
```java
   @Bean
   public MybatisPlusInterceptor mybatisPlusInterceptor() {
       MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();   
       //分页插件
       interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.H2));
       return interceptor;
   }
```

2. 测试
```java
    @Test
    public void testSelect() {
        Page<User> page=new Page<>(1,2);
        userMapper.selectPage(page,null);
        page.getRecords().forEach(System.out::println);
    }
```

### 逻辑删除
物理删除 ：从数据库中直接移除
逻辑删除:    数据库中没有被移除，而是通过一个变量来让他失效！deleted = 0r> deleted = 1

1. 字段上加上@TableLogic注解
```java
@TableLogic
private Integer deleted;
```

2. 配置
```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: flag # 全局逻辑删除的实体字段名(since 3.3.0,配置后可以忽略不配置步骤2)
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```

3. 测试
```java
    @Test
    public void testLogicSql(){
        userMapper.deleteById(1L);
    }
```

### 性能分析插件



### 条件构造器
使用wrapper代替复杂sql
```java
    @Test
    public void testWrapper(){
        QueryWrapper<User> wrapper=new QueryWrapper<>();
        wrapper.isNotNull("name")
                .isNotNull("id");
        List<User> list=userMapper.selectList(wrapper);
        list.forEach(System.out::println);
    }
```
|查询方式|	说明|
|----|----|
|setSqlSelect |设置 SELECT 查询字段|
|where |	WHERE 语句，拼接 + WHERE 条件|
|and|	AND 语句，拼接 + AND 字段=值|
|andNew|	AND 语句，拼接 + AND (字段=值)|
|or|	OR 语句，拼接 + OR 字段=值|
|orNew|	OR 语句，拼接 + OR (字段=值)|
|eq|	等于=|
|allEq|	基于 map 内容等于=|
|ne|	不等于<>|
|gt|	大于>|
|ge|	大于等于>=|
|lt|	小于<|
|le|	小于等于<=|
|like|	模糊查询 LIKE|
|notLike|	模糊查询 NOT LIKE|
|in|	IN 查询|
|notIn|	NOT IN 查询|
|isNull|	NULL 值查询|
|isNotNull|	IS NOT NULL|
|groupBy|	分组 GROUP BY|
|having|	HAVING 关键词|
|orderBy|	排序 ORDER BY|
|orderAsc|	ASC 排序 ORDER BY|
|orderDesc|	DESC 排序 ORDER BY|
|exists|	EXISTS 条件语句|
|notExists|	NOT EXISTS 条件语句|
|between|	BETWEEN 条件语句|
|notBetween|	NOT BETWEEN 条件语句|
|addFilter|	自由拼接 SQL|
|last	| 拼接在最后，例如：last(“LIMIT 1”)|



### 代码生成器

[使用配置 | MyBatis-Plus (baomidou.com)](https://baomidou.com/pages/56bac0/)

