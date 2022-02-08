# MYSQL常用shell命令
```mysql shell
	$ mysql -u root -p  # 进入MySQL bin目录后执行，回车后输入密码连接。
                        # 常用参数：-h 服务器地址，-u 用户名，-p 密码，-P 端口
 #数据库操作
 > create database dbname;     # 创建数据库，数据库名为dbname
 > CREATE DATABASE `todo` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;              #创建todo数据库，并指定字符集
 > show databases;                            # 显示所有数据库
 > alter database character set utf8;         # 修改数据库字符集
 > use dbname;                                # 选择数据库
 > status;                                    # 查看当前选择（use）的数据库
 > drop database dbname;                      # 删除数据库
 #数据表操作
> show tables;                               # 显示所有表
> describe tablename;                        # 表结构详细描述
> desc tablename;                            # 同 describe 命令一样
> create table newtable like oldtable;       # 复制表结构
> insert into newtable select * from oldtable;  #复制表数据
> rename table tablelname to new_tablelname  # 重命名表，同时命名多个表用逗号“,”分割
> drop table tablename;                      # 删除表
> select version(),current_date;             # 显示当前mysql版本和当前日期
#修改rootmima
$ mysqladmin -u root password                     # 原始密码为空的情况
New password: <输入新的密码>
Confirm new password: <再次输入新密码>
$ mysqladmin -u root -p password                  # 原始密码不为空的情况
Enter password: <输入旧的密码>
New password: <输入新的密码>
Confirm new password: <再次输入新密码>
$ mysqladmin -uroot -p123456 password             # 原始密码不为空的情况，效果和第二种方法一样，只是显式的输入了原始密码
New password: <输入新的密码>
Confirm new password: <再次输入新密码>

```


# 环境变量配置
PC: classpath=.;jar包mulu;
IDEA:将jar包添加为项目库

# JDBC编程六步
1. 注册驱动（作用:告诉Java程序，即将要连接的是哪个品牌的数据库)
2. 获取连接（表示Jvm的进程和数据库进程之间的通道打开了，这属于进程之间的通信，重量级的，使用完之后一定要关闭)
3. 获取数据库操作对象(专门执行sql语句的对象)
4. 执行sql语句(DQL DML. . . . )
5. 处理查询结果集（只有当第四步执行的是select语句的时候，才有这第五步处理查询结果集。)
6. 释放资源（使用完资源之后一定要关闭资源。Java和数据库属于进程间的通信，)

```java
public class JDBCTest01 {
    public static void main(String[] args){
        Connection connection=null;
        Statement statement=null;
        ResultSet resultSet=null;
        try {
            //1. 注册驱动
            Driver driver = new com.mysql.jdbc.Driver();//多态,父类型引用指向子类型对象
            DriverManager.registerDriver(driver);
            //2.获取链接
            String url="jdbc:mysql://127.0.0.1:3306/jdbctest";
            String user="root";
            String password="123456";
            connection=DriverManager.getConnection(url,user,password);
            //3. 获取数据库执行对象
            statement=connection.createStatement();
            //执行SQL语句
            String sql="insert into person values(1,'lqw')";
            int count=statement.executeUpdate(sql);
            System.out.println(count);
            // 5. 处理数据集
            String sql_select="select * from person; ";
            resultSet=statement.executeQuery(sql_select);
            while (resultSet.next()){
                //参数为查询结果中的列名(别名)
                String id=resultSet.getString("id");
                String name=resultSet.getString("name");
                System.out.println(id+name);
            }
        }catch(SQLException e){
            e.printStackTrace();
        }finally {
            //6. 释放资源 从小到大 分别try...catch
            try{
                if(resultSet!=null)
                    resultSet.close();
            }catch (SQLException e){
                e.printStackTrace();
            }
            try{
                if(statement!=null)
                    statement.close();
            }catch(SQLException e){
                e.printStackTrace();
            }
            try{
                if(connection!=null)
                    connection.close();
            }catch(SQLException e){
                e.printStackTrace();
            }
        }
        //注册驱动方式二:类加载器(常用)
        //参数是字符串 可以写到配置文件中
        try {
            Class.forName("com.mysql.jdbc.Driver");
            Connection connection1=DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/jdbctest","root","123456");
            System.out.println(connection1);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e){
            e.printStackTrace();
        }

    }
}
```
# SQL注入
SQL 注入就是在用户输入的字符串中加入 SQL 语句，如果在设计不良的程序中忽略了检查，那么这些注入进去的 SQL 语句就会被数据库服务器误认为是正常的 SQL 语句而运行，攻击者就可以执行计划外的命令或访问未被授权的数据。
## 原理:
1）恶意拼接查询
2）利用注释执行非法命令
3）传入非法参数
4）添加额外条件
## 解决办法
让用户提供的信息不参与SQL语句编译过程
```java
 Connection connection1=null;
        PreparedStatement preparedStatement=null;
        ResultSet resultSet1=null;
        try {
            //注册驱动
            Class.forName("com.mysql.jdbc.Driver");
            //获取连接
            connection1=DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/jdbctest","root","123456");
            System.out.println(connection1);
            //获取预编译的数据库操作对象
            //一个?表示一个占位符,编译时接收一个值
            String presql="select * from jdbctest where id=?";
            preparedStatement=connection1.prepareStatement(presql);
            //给占位符?传值 (下标,值) jdbc中所有下标从1开始
            preparedStatement.setInt(1,1);
            //执行sql
            resultSet1=preparedStatement.executeQuery();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e){
            e.printStackTrace();
        }
```
# JDBC中事务
## 开启事务
JDBC中事务默认为自动提交
`connection.setAutoCommit(false);//关闭自动提交机制`
`connection.commit();//手动提交`
`connection.rollback();//手动回滚`

## 锁机制
### 从数据库系统角度
#### 排他锁(写锁)
exclusive locks (X锁),事务T对数据对象加上X锁,只允许T读取和修改A,任何其他事务不能对A加任何类型的锁,直到T释放A上的锁.
#### 共享锁(读锁)
share locks(S锁),事务T对数据对象A加上S锁,则事务T可以读A但不能修改A,其他事务只能再对A加S锁,不能加X锁,直到释放A上所有的S锁.
#### 更新锁
(U锁)：用来预定要对此页施加X锁，它允许其他事务读，但不允许再施加U锁或X锁；当被读取的页将要被更新时，则升级为X锁；U锁一直到事务结束时才能被释放。
#### 封锁协议
* 一级封锁协议
一级封锁协议是指，事务T在修改数据R之前必须先对其加X锁，直到事务结束才释放。一级封锁协议可以防止丢失修改，并保证事务T是可恢复的。
* 二级封锁协议
二级封锁协议是指，在一级封锁协议基础上增加事务T在读数据R之前必须先对其加S锁，读完后即可释放S锁。二级封锁协议出防止了丢失修改，还可以进一步防止读“脏”数据。
* 三级封锁协议
三级封锁协议是指，在一级封锁协议的基础上增加事务T在读数据R之前必须先对其加S锁，直到事务结束才释放。三级封锁协议出防止了丢失修改和读“脏”数据外，还可以进一步防止了不可重复读。
### 从程序员角度
#### 悲观锁
悲观锁（Pessimistic Lock）：具有强烈的独占和排他特性。它指的是对数据被外界（包括本系统当前的其他事务，以及来自外部系统的事务处理）修改持保守态度，因此，在整个数据处理过程中，将数据处于锁定状态。悲观锁的实现，往往依靠数据库提供的锁机制（也只有数据库层提供的锁机制才能真正保证数据访问的排他性，否则，即使在本系统中实现了加锁机制，也无法保证外部系统不会修改数据）。
使用悲观锁:

```sql
autocommit=0;//关闭自动提交
begin;//开启事务
select * from ? for update;//加锁
commit;//提交同时释放锁
```
锁级别:
   * 若明确指明主键，且结果集有数据，行锁；
   * 若明确指明主键，结果集无数据，则无锁；
   * 若无主键，且非主键字段无索引，则表锁；
   *  若使用主键但主键不明确，则使用表锁；
#### 乐观锁
乐观锁，大多是基于数据版本  (Version)记录机制实现。何谓数据版本？即为数据增加一个版本标识，在基于数据库表的版本解决方案中，一般是通过为数据库表增加一个 “version” 字段来 实现。 读取出数据时，将此版本号一同读出，之后更新时，对此版本号加一。此时，将提 交数据的版本数据与数据库表对应记录的当前版本信息进行比对，如果提交的数据 版本号大于数据库表当前版本号，则予以更新，否则认为是过期数据。

### 活锁
活锁概念类似于操作系统中的饥饿
### 死锁
概念与操作系统中一致
* 预防死锁:(1)一次封锁法;(2)顺序封锁法
* 死锁的诊断与解除:(1)超时法;(2)等待图法

## 隔离级别
并发下事务会产生的问题:脏读/不可重复读/幻读
1、DEFAULT
默认隔离级别，每种数据库支持的事务隔离级别不一样，如果Spring配置事务时将isolation设置为这个值的话，那么将使用底层数据库的默认事务隔离级别。顺便说一句，如果使用的MySQL，可以使用"select @@tx_isolation"来查看默认的事务隔离级别

2、READ_UNCOMMITTED
读未提交，即能够读取到没有被提交的数据，所以很明显这个级别的隔离机制无法解决脏读、不可重复读、幻读中的任何一种，因此很少使用

3、READ_COMMITED
读已提交，即能够读到那些已经提交的数据，自然能够防止脏读，但是无法限制不可重复读和幻读

4、REPEATABLE_READ
重复读取，即在数据读出来之后加锁，类似"select * from XXX for update"，明确数据读取出来就是为了更新用的，所以要加一把锁，防止别人修改它。REPEATABLE_READ的意思也类似，读取了一条数据，这个事务不结束，别的事务就不可以改这条记录，这样就解决了脏读、不可重复读的问题，但是幻读的问题还是无法解决

5、SERLALIZABLE
串行化，最高的事务隔离级别，不管多少事务，挨个运行完一个事务的所有子事务之后才可以执行另外一个事务里面的所有子事务，这样就解决了脏读、不可重复读和幻读的问题了
