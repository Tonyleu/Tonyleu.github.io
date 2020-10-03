---
title: JDBC
---

## JDBD

### 1.概念：Java Database Connectivity （Java数据库连接）Java语言操作数据库

- JDBC本质：其实是官方（sun公司）定义的一套操作所有关系型数据库的规则，即接口。  
- 各个数据库厂商实现这套接口，提供数据库驱动jar包。  
- 我们可以使用这套接口（JDBC）编程，真正执行的代码是驱动jar包中的实现类。  

### 2.快速入门：
 
1. 导入驱动jar包。mysql-connector-java-8.0.12.jar  
	- 复制mysql-connector-java-8.0.12.jar到项目的lib目录下  
    - 右键->add as library  
2. 注册驱动  
3. 获取数据库的连接对象 connection  
4. 定义sql  
5. 获取执行sql语句的对象 statement  
6. 执行sql，接收返回结果  
7. 处理结果  
8. 释放资源  
```java
package pers.liangxionghua.JDBC.JDBCDemo;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;

public class JDBCDemo1 {
    public static void main(String[] args) {
        //1.导入驱动jar包
        Connection connection = null;
        Statement statement = null;
        try {
            //2.注册驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            //3.获取数据库的连接对象
            connection = DriverManager.getConnection(
            	"jdbc:mysql://localhost:3306/mydbTest?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true"
            	, "root"
            	, "root");
            //4.定义sql语句
            String sql = "update usertable set userPassword = '1234' where userName = '123'";
            //5.获取执行sql语句的对象 statement
            statement = connection.createStatement();
            //6.执行qsl
            int i = statement.executeUpdate(sql);
            //7.处理结果
            System.out.println(i);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            try{
                if (statement != null) {
                    statement.close();
                }
                if (connection != null) {
                    connection.close();
                }
            } catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

### 3.详解各个对象：

1. DriverManager：驱动管理对象  
	- 注册驱动 public static void registerDriver(Driver driver)向 DriverManager 注册给定驱动程序。  
		新加载的驱动程序类应该调用 registerDriver 方法让 DriverManager 知道自己。  
		写代码使用：Class.forName("com.mysql.cj.jdbc.Driver");  
		通过查看源码发现：在**com.mysql.cj.jdbc.Driver**类中存在静态代码块  
```java
static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
```
		**注意：mysql-5之后的驱动jar包可以省略注册驱动的步骤。**  
	- 获取数据库连接  
			- 方法：static Connection getConnection(String url, String user, String password) 试图建立到给定数据库 URL 的连接。  
			- 参数：  
				* url：指定连接的路径  
					* 语法：jdbc:mysql://ip地址（域名）：端口号/数据库名称?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true  
					* 例子：jdbc:mysql://localhost:3306/mydbTest?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true  
					* 细节：如果连接的是本机的mysql服务器，并且端口号是3306，  
						则url可以简写为jdbc:mysql:///mydbTest?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true  
				* user：用户名  
				* password：密码  
2. Connection：数据库连接对象  
	1. 功能：  
		1. 获取执行sql的对象  
			* Statement createStatement() 创建一个 Statement 对象来将 SQL 语句发送到数据库。   
			* PreparedStatement prepareStatement(String sql) 创建一个 PreparedStatement 对象来将参数化的 SQL 语句发送到数据库。   
		2. 管理事务：  
			* 开启事务：void setAutoCommit(boolean autoCommit) 调用该方法设置参数为false，即开启事务。  
			* 提交事务：void commit() 使所有上一次提交/回滚后进行的更改成为持久更改，并释放此 Connection 对象当前持有的所有数据库锁。  
			* 回滚事务：void rollback() 取消在当前事务中进行的所有更改，并释放此 Connection 对象当前持有的所有数据库锁。   
3. Statement：执行sql的对象  
	1. 执行sql  
		1. boolean execute(String sql)  可以执行任意sql，了解即可。  
		2. boolean execute(String sql)  执行DML（insert，delete，update）语句，DDL（creat，alter，drop）语句（不经常使用）  
			* 返回值：影响的行数  
		3. ResultSet executeQuery(String sql) 执行DQL（select）语句  
	2. 练习  
		1. usertable表 添加一条记录  
		2. usertable表 修改记录  
		3. usertable表 删除一条记录  
4. ResultSet：结果集对象  
	* boolean next() ：游标向下移动一行，判断当前行是否是最后一行末尾（是否有数据），如果是，则返回false，如果不是则返回true  
	* getXxx（参数）：获取数据  
		* Xxx：代表数据类型 如：getInt（），getString（）  
		* 参数：  
			1. int:代表列的编号，从1开始 如：getString（1）  
			2. String：代表列的名称 如：getString（"userName"）  
	* 注意：  
		* 使用步骤：  
			1. 游标向下移动一行  
			2. 判断是否有数据  
			3. 获取数据  
	* 练习：  
		1. 定义User类  
		2. 定义方法 public List<User> findAll(){}  
		3. 实现方法 select * from usertable;  
5. PreparedStatement：执行sql的对象  
	1. SQL注入问题:在拼接SQL时，有一些SQL的关键字参与拼接。会造成安全问题。  
		1.输入用户随便，输入密码：a' or 'a' = 'a  
		2.sql:select * from usertable where userName = 'sadsada' and userPassword = 'a' or 'a' = 'a'  
	2. 解决SQL注入问题：使用PreparedStatement对象来解决  
	3. 预编译SQL：参数使用?作为占位符  
	4. 步骤：  
		1. 导入驱动jar包。mysql-connector-java-8.0.12.jar  
		2. 注册驱动  
		3. 获取数据库的连接对象 connection  
		4. 定义sql  
			* 注意：sql的参数使用?作为占位符。如：select * from usertable where userName = ? and userPassword = ?  
		5. 获取执行sql语句的对象 PreparedStatement prepareStatement(String sql)   
		6. 给?赋值  
			* 方法：setXxx(参数1,参数2)  
				* 参数1：?的位置编号，从1开始  
				* 参数2：?的值  
		7. 执行sql，接收返回结果，不需要传递sql语句  
		8. 处理结果  
		9. 释放资源  
	5. 注意：后期都会使用PreparedStatement来完成增删改查的所有操作  
		1. 可以防止SQL注入  
		2. 效率更高  

### 抽取JDBC工具类：JDBCUtils

* 目的：简化书写  
* 分析：  
	1. 抽取注册驱动  
	2. 抽取一个方法获取连接对象  
		* 需求：不想传递参数（麻烦），还得保证工具类的通用性  
		* 解决：配置文件  
			jdbc.properties  
				url=  
				user=  
				password=  
	3. 抽取一个方法释放资源  
* 练习  
	* 需求  
		1. 通过键盘录入用户名与密码  
		2. 判断用户是否登录成功  
```sql
	select * from usertable where username = ‘’ and password = ‘’；
```  
	* 步骤  
		1. 创建数据库表usertable  

### JDBC控制事务

* 事务：一个包含多个步骤的业务操作。如果这个业务操作被事务管理，那么这多个步骤要么同时成功，要么同时失败。  
* 操作：  
	1. 开启事务  
	2. 提交事务  
	3. 回滚事务  
* 使用 **Connection** 对象来管理事务  
	* 开启事务：**void setAutoCommit(boolean autoCommit)** 调用该方法设置参数为**false**，即开启事务。   
		* 在执行sql之前开启事务  
	* 提交事务：**void commit()** 使所有上一次提交/回滚后进行的更改成为持久更改，并释放此 **Connection** 对象当前持有的所有数据库锁。   
		* 当所有sql执行完提交事务  
	* 回滚事务：**void rollback()** 取消在当前事务中进行的所有更改，并释放此 **Connection** 对象当前持有的所有数据库锁。   
		* 在**catch**中回滚事务  