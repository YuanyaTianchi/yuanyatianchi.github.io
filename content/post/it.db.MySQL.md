

+++

title = "MySQL"
description = "MySQL"
tags = ["it", "db"]

+++



# mysql

https://www.mysql.com/ ，https://www.mysql.com/cn/ 

- 3306
- sql
  - DDL，数据定义语言。定义和管理数据对象，如数据库，数据表等。CREATE、DROP、ALTER
  - DML，数据操作语言。用于操作数据库对象中所包含的数据。| NSERT、UPDATE、DELETE
  - DQL，数据查询语言。用于查询数据库数据。SELECT
  - DCL，数据控制语言。用来管理数据库的语言，包括管理权限及数据更改。GRANT、COMMIT、ROLLBACK

## client

### go

```shell
$ go get -u github.com/go-sql-driver/mysql
```

##### 连接

###### db/db.go

```go
import (
	"database/sql"
	_ "github.com/go-sql-driver/mysql" //mysql驱动，不使用其提供的方法，通过'_'加载其初始化函数即可
)

var (
	/*DB是一个数据库(操作)句柄，代表一个具有零到多个底层连接的连接池，它可以安全的被多个goroutine同时使用。
	sql包会自动创建和释放连接，它也会维护一个闲置连接的连接池。*/
	MyDB *sql.DB
	err  error
)

func init() {
	/*Open：创建一个DB实例。Open过程只验证参数，不参创建与数据库的连接，自然也不会校验账号密码数据源名称中账号密码等数据正确性*/
	MyDB, err = sql.Open("mysql", "root:yuanya@tcp(192.168.31.108:3306)/yuanya")
	if err != nil {
		panic(err)
	}
	defer MyDB.Close()

	/*配置
	MyDB.SetMaxOpenConns(0)    //设置最大连接数，默认为0(<=0时无限制)
	MyDB.SetConnMaxLifetime(0) //设置最大闲置连接数，默认为0(<=0时不保留闲置连接)
	MyDB.SetMaxIdleConns(-1)   //设置连接池大小，默认为-1(<=0时)*/

	/*尝试与数据库建立连接。将检查数据源的名称是否合法*/
	if err := MyDB.Ping(); err != nil {
		panic(err.Error())
	}
}
```

##### 操作

###### dao/UserDao.go

```go
type User struct {
	Id       int
	Username string
	Password string
}

/*row：查询一行结果*/
func getById(id int) (user *User, err error) {
	sql := "select * from user where id=?"

	/*sql拼接*/
	row := db.MyDB.QueryRow(sql, id) //QueryRow返回一行结果

	err = row.Scan(&user.Id, &user.Username, &user.Password) //Scan将该行查询结果各列分别保存进dest参数指定的值中
	return
}

/*rows：查询多行结果*/
func listByUsernameLike(usernameLike string) (users []*User, err error) {
	sql := "select * from user where username like ?"

	/*sql拼接
	rows, err := db.MyDB.Query(sql, usernameLike) //Query返回多行结果
	if err != nil {
		fmt.Println(err)
	}*/

	/*sql预处理。任何时候都不应该拼接SQL语句，而是要进行预处理，以防SQL注入(如"or 1=1#"等)*/
	stmt, err := db.MyDB.Prepare(sql)
	if err != nil {
		fmt.Println("Prepare异常", err)
	}
	rows, err := stmt.Query(usernameLike) //stmt方法与db基本一致，只是不用再传入sql
	if err != nil {
		fmt.Println("Query异常", err)
	}

	/*迭代遍历行*/
	for rows.Next() {
		user := User{}
		rows.Scan(&user.Id, &user.Username, &user.Password)
		users = append(users, &user)
	}
	return
}

/*增*/
func insert(user *User) (err error) {
	sql := "insert into user(username, password) value(?,?)"
	stmt, err := db.MyDB.Prepare(sql)
	if err != nil {
		fmt.Println("Prepare异常", err)
	}
	_, err = stmt.Exec(user.Username, user.Password) //执行
	if err != nil {
		fmt.Println("Exec异常", err)
	}
	return
}

/*事务。db.Begin开启事务，tx.Rollback()回滚事务，tx.Commit()提交事务
如果数据库具有单连接状态的概念，该状态只有在事务中被观察时才可信。一旦调用BD.Begin,返回的Tx会绑定到单个连接
当调用事务Tx的Commit或Rollback后，该事务使用的连接会归还到DB的闲置连接池中*/
func tx() {
	tx, err := db.MyDB.Begin() //开启事务
	if err != nil {
		if tx != nil {
			tx.Rollback() //回滚事务
		}
		fmt.Printf("begin trans failed, err:%v\n", err)
		return
	}

	sql1 := "Update user set age=30 where id=?"
	ret1, err := tx.Exec(sql1, 2)
	if err != nil {
		tx.Rollback()
		fmt.Printf("exec sql1 failed, err:%v\n", err)
		return
	}

	affRow1, err := ret1.RowsAffected()
	if err != nil {
		tx.Rollback()
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}

	sql2 := "Update user set age=40 where id=?"
	ret2, err := tx.Exec(sql2, 3)
	if err != nil {
		tx.Rollback()
		fmt.Printf("exec sql2 failed, err:%v\n", err)
		return
	}

	affRow2, err := ret2.RowsAffected()
	if err != nil {
		tx.Rollback()
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}

	fmt.Println(affRow1, affRow2)
	if affRow1 == 1 && affRow2 == 1 {
		fmt.Println("事务提交啦...")
		tx.Commit() //提交事务
	} else {
		tx.Rollback()
		fmt.Println("事务回滚啦...")
	}

	fmt.Println("exec trans success!")
}
```

```sql
CREATE TABLE user(
	id INT PRIMARY KEY AUTO_INCREMENT,
	username VARCHAR(15) UNIQUE NOT NULL,
	password VARCHAR(15) NOT NULL,
)ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
```

### java

- jdbc

```java
public class JdbcDemo {
    static final String DRIVER = "com.mysql.jdbc.Driver";
    static final String URL = "jdbc:mysql://localhost:3306/yuanya_mybatis?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true";
    static final String USERNAME = "root";
    static final String PASSWORD = "root";

    @Test
    public void StatementDemo() {
        //...
    }
    @Test
    public void PreparedStatementDemo() {
        //...
    }
    @Test
    public void batchDemo() {
        //...
    }
}
```

###### Statement

无预编译，每次需要经历完整的过程，都较慢

```java
@Test
public void QueryStatementDemo() {
    Connection conn = null;
    Statement stmt = null;
    List<User> users = new ArrayList();
    try {
        //STEP 2: 注册mysql的驱动
        Class.forName(DRIVER);

        //STEP 3: 获得一个连接
        conn = DriverManager.getConnection(URL, USERNAME, PASSWORD);

        //STEP 4: 创建一个查询
        String name = "yuanya";
        String sql = "SELECT * FROM user WHERE name='" + name + "'"; //加'防sql注入
        stmt = conn.createStatement(); //Statement
        ResultSet rs = stmt.executeQuery(sql); //执行查询，获得结果集

        //STEP 5: 从结果集中获取数据并转化成bean
        while (rs.next()) {
            User user = new User();
            user.setId(rs.getInt("id"));
            user.setName(rs.getString("name"));
            user.setAge(rs.getInt("age"));
            user.setDeleteFlag(rs.getString("delete_flag"));
            users.add(user);
        }

        // STEP 6: 关闭连接
        rs.close();
        stmt.close();
        conn.close();
    } catch (SQLException se) {
        // Handle errors for JDBC
        se.printStackTrace();
    } catch (Exception e) {
        // Handle errors for Class.forName
        e.printStackTrace();
    } finally {
        // finally block used to close resources
        try {
            if (stmt != null)
                stmt.close();
        } catch (SQLException se2) {
            // nothing we can do
        }
        try {
            if (conn != null)
                conn.close();
        } catch (SQLException se) {
            se.printStackTrace();
        }
    }
}
```

###### PreparedStatement

有预编译，所以第一次查询很慢，但是之后很快

```java
@Test
public void QueryPreparedStatementDemo() {
    Connection conn = null;
    PreparedStatement stmt = null;
    List<User> users = new ArrayList();
    try {
        // STEP 2: 注册mysql的驱动
        Class.forName(DRIVER);

        // STEP 3: 获得一个连接
        conn = DriverManager.getConnection(URL, USERNAME, PASSWORD);

        // STEP 4: 创建一个查询
        String sql = "SELECT * FROM user WHERE name=? "; //占位符，有预处理，可以避免sql注入
        stmt = conn.prepareStatement(sql); //
        stmt.setString(1, "yuanya or 1=1");
        /*得到的sql将是"SELECT * FROM user WHERE name='yuanya or 1=1'"而不是"SELECT * FROM user WHERE name=yuanya or 1=1"
            拼接到sql中的字符串变量将作为一个整体'yuanya or 1=1'，从而防止sql注入*/
        ResultSet rs = stmt.executeQuery();

        // STEP 5: 从resultSet中获取数据并转化成bean
        while (rs.next()) {
            User user = new User();
            user.setId(rs.getInt("id"));
            user.setName(rs.getString("name"));
            user.setAge(rs.getInt("age"));
            user.setDeleteFlag(rs.getString("delete_flag"));
            users.add(user);
        }

        // STEP 6: 关闭连接
        rs.close();
        stmt.close();
        conn.close();
    } catch (SQLException se) {
        // Handle errors for JDBC
        se.printStackTrace();
    } catch (Exception e) {
        // Handle errors for Class.forName
        e.printStackTrace();
    } finally {
        // finally block used to close resources
        try {
            if (stmt != null)
                stmt.close();
        } catch (SQLException se2) {
            // nothing we can do
        }
        try {
            if (conn != null)
                conn.close();
        } catch (SQLException se) {
            se.printStackTrace();
        }
    }
}
```

###### batch操作

批量操作，只连接了一次数据库

```java
@Test
public void batchDemo() {
    Connection conn = null;
    PreparedStatement stmt = null;
    try {
        // STEP 2: 注册mysql的驱动
        Class.forName(DRIVER);

        // STEP 3: 获得一个连接
        conn = DriverManager.getConnection(URL, USERNAME, PASSWORD);

        // STEP 4: 启动手动提交。写操作需要用到事务，读操作不需要
        conn.setAutoCommit(false);

        // STEP 5: 创建一个更新
        String sql1 = "UPDATE user SET delete_flag= '1' WHERE name='yuanya'";
        String sql2 = "UPDATE user SET delete_flag= '1' WHERE name='tianchi'";
        stmt.addBatch(sql1);
        stmt.addBatch(sql2);
        System.out.println(stmt.toString()); //打印sql
        int[] ret = stmt.executeBatch(); //影响行数是一个int[]，对应每个sql
        System.out.println("影响数据库行数: " + ret);

        // STEP 6: 手动提交数据
        conn.commit();

        // STEP 7: 关闭连接
        stmt.close();
        conn.close();
    } catch (SQLException se) {
        // Handle errors for JDBC
        try {
            conn.rollback();
        } catch (SQLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        se.printStackTrace();
    } catch (Exception e) {
        try {
            conn.rollback();
        } catch (SQLException e1) {
            // TODO Auto-generated catch block
            e1.printStackTrace();
        }
        e.printStackTrace();
    } finally {
        // finally block used to close resources
        try {
            if (stmt != null)
                stmt.close();
        } catch (SQLException se2) {
        }// nothing we can do
        try {
            if (conn != null)
                conn.close();
        } catch (SQLException se) {
            se.printStackTrace();
        }
    }
}
```



## shell

```mysql
#查看最大连接数
show variables like '%max_connections%'; 

#设置全局最大连接数
set GLOBAL max_connections = 999;
#列出全局变量
show global variable

#列出系统会话变量
show session variable

#定义变量
declare 变量名1,变量名2... 类型1,类型2... [default 值]
```



### database

- mysql初始database：`information_schema`、`sysql`、`perfomance_schema`

```mysql
#列出
show databases
#创建
create database 库名
#删除
drop database 库名
#连接
use 库名
```



### table

- 操作表之前需要先通过 use 命令选择要操作的数据库

```sql
#列出
show tables
#创建信息
show create table 表名
#列信息
show columns from 表名
#索引信息
show index from 表名

#创建
create table 表名(列名1 数据类型(长度) 约束1 约束2, 列名2 数据类型(长度) 约束1 约束2...);
create table `doctor_patient_group` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `created_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  key `idx_created_at` (`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='hahah';;
#复制表
create table 表名2 as(select * from 表名1)
#复制表结构
create table 表名2 like 表名1

#修改表名
rename table 旧表名 to 新表名
alter table 旧表名 rename as 新表名
#修改表的属性
alter table 表名 change 旧列名 新列名 数据类型(长度) 约束1 约束2
alter table 表名 add  column 列名 数据类型(长度);
alter table 表名 drop  column 列名;

#删除
dorp table 表名
```

```mysql
#查看Mysql数据库管理系统的性能及统计信息
show table status like [FROM db_name] [LIKE 'pattern'] \G;
```

##### 数据类型

###### MySQL常用数据类型

- 字符串型 ：VARCHAR(varchar(10)：可变长度的字符串"abc")、 CHAR(char(10)：固定长度的字符串"abc          ")
- 大数据类型：BLOB（存储大的二进制数据）、TEXT（存储海量的文本数据）
- 数值型：TINYINT 、SMALLINT、INT、BIGINT、FLOAT、DOUBLE
- 逻辑性 ：BIT
- 日期型：DATE、TIME、DATETIME、TIMESTAMP

**字段约束**

- 列名 数据类型 约束1 约束2
- 非空约束：not null
- 唯一约束：unique
- 主键约束：primary key，非空，唯一，alter table tablename drop primary key删除主键
- 主键自增：auto_increment
- 外键添加：
  - 列名 类型(长度) [constraint fk_外键名] references 表名2(外键名)
  - [constraint fk_外键名] foreign key(外键名) references 表名2(外键名)
  - slter table 表名 add [constraint kf_外键名] foreign ke(外键名) reference 表名2(外键名) 



### crud

#### select

```mysql
#查询所有*
select * from 表名
#查询指定列
select 列名1, 列名2, 函数名3(列名3)... from 表名
```

###### as

```mysql
#查询字段定义别名as，as可以省略
select 列名 as '别名' from 表名
```

###### 聚合函数

```mysql
#聚合函数。sum()和，avg()均值，count()个数，max()最大值，min()最小值
select 列名, 函数名(列名) as '别名' from 表名
```

###### distinct

```mysql
#去重查询distinct。distinct对其后列名去重
select distinct 列名 from 表名
```

###### limit

```mysql
#分页查询limit。分页查询，index从0开始
select * from 表名 limit index,pageSize
```

###### where

```mysql
#条件查询where
select * from 表名 where 条件式
```

- where：where 条件式
  - 日期：WHERE date<'999-9-9'，日期date在999年9月9日之前
  - and：where num>=3 and num<=9，num在3-9之间
  - or：WHERE num=10 OR num=30，num为3或者为9
  - not：WHERE NOT(num>2000)，num不大于2000
  - between...and：where sal between3 AND 9，num在3-9之间
  - in(值1,值2...)：where num in(3,9...)，num为3或9
  - length(列名)：where length(name)=5，名字长度为5

```mysql
#模糊查询like，_代替一个字符，%代替任意长度字符，转义字符有效
select * from 表名 where 列名 like '_xxx%'
```

###### group

```mysql
#分组查询，group by
group by 列名
#分组查询，group by ...... having
group by 列名 having 条件式
```

###### order

```mysql
#排序查询，order by，默认升序，asc升序，desc降序
order by 列名1 desc, 列名2 asc
```

###### 顺序小结

```mysql
#顺序小结：(S-F-W-G-H-O) select ... from ... where ... group by... having... order by ... ; 
SELECT id,count(*) AS '个数'
FROM user
WHRER name LIKE '_xxx%' #筛选在分组之前
GROUP BY  rank          #分组在再筛选之前，因为分组得到的数据count被再筛选使用
HAVING count(*)>1       #再筛选：having对分组之后的数据进行再筛选
ORDER BY count(*) DESC; #排序在最后，因为筛选完才能排序
```

###### 关联查询

```mysql
#内连接where，有无外键约束查询都有效，对删改有约束效果
where：select 列名1,列名2... from 表名1,表名2 where 表名1.列名=表名2.列名 and empno=7788
#内连接inner join
select 列名1,列名2... from 表名1 inner join 表名2 on 表名1.列名=表名2.列名 and empno=7788

#左外连接left outer join。学生表(student)、必修表(required)、选修表(elective)，查询学生的id、name、选修科目数量、必修科目数量、科目总数
select s.student_id,s.student_name, ifnull(r.rCount,0) as rCount, ifnull(e.eCount,0) as eCount, ifnull(r.rCount,0)+ifnull(e.eCount,0)*1 as amount 
from student as s 
left join ( 
	select student_id, count(1) as rCount 
	from required 
	group by student_id 
) as r on s.student_id = r.student_id 
left join (
	select student_id,count(1) as eCount 
	from elective 
	group by student_id 
) e on p.student_id = e.student_id 
where r.rCount>0 or e.eCount>0 
order by activity desc 
limit 0,9;

#右外连接：lright outer join
#全外连接：full outer join(mysql不支持)
```

###### 嵌套查询

外层的查询块称为父查询，内层的查询块称为子查询

###### 关联子查询

子查询不可以单独运行，依赖于福查询中的数据	

```mysql
#exists。exists(子查询)返回true或false，为true则查询符合子查询条件的父查询内容
select * from 表1 where exists(select * from 表名2 where 表1.外键名=外键名)
```

###### 非关联子查询

子查询可以单独运行

```mysql
#单行单列子查询：子查询结果返回的值只有一个，< 小于，> 大于，<= 小于等于，>=大于等于，=等于，!=或<>不等于
select 列名1 from 表名1 where 列名<(select 函数名(列名) from 表名2 where 条件式; );
select student_id from score where count(course_i)

#单列多行子查询：子查询结果返回的值有多个。any、all、in、not in
select 列名1 from 表名1 where 列名 > any (select 函数名(列名) from 表名2 where 条件式; );
select 列名1 from 表名1 where 列名 < all (select 函数名(列名) from 表名2 where 条件式; );
select 列名1 from 表名1 where 列名 in (select 函数名(列名) from 表名2 where 条件式; );
#查询参加算计考试且不参加英语和数学考试的学生的所有信息
select t
where id in(
    select stu_id
    from scor
    where c_name='计算机' and stu_id not in(
        select stu_id
        from score
        where c_name='英语' or c_name='数学'
    )
)

#多行多列子查询：子查询的结果是一个表
```

###### 合并查询

```mysql
#合并查询：union(去重合并)，union all(无去重合并)
select 列名1,列名3 from 表1 union select 列名2,列名4 from 表2
```





#### insert

- 存在的判断，需要插入内容包含主键或唯一索引（字段只要被UNIQUE修饰就有唯一索引），且3种操作无论是否成功插入都会使自增的主键+1

```mysql
#如果已存在则报错，如果不存在则插入
insert into 表名(列名1, 列名2...) values (值1,值2,...),(值1,值2,...);
#已存在行则忽略，不存在行则插入
insert ignore into ...
#已存在行则替换，不存在行则插入
replace into ...
```

#### update

```mysql
update 表名 set 列名1=值1, 列名2=值2... where 条件式
```

#### delete

```mysql
delete from 表名 where 条件式
```



### 视图

一个或多个表的整个或部分的映射，一般只用来查询

```mysql
#创建
create view 视图名 as 查询语句
```



### 存储过程

无返回值，但有输出参数，call调用

```mysql
#创建函数(in，out，inout)
create procedure 函数名( in 输入参数名 varchar(18), out 输出参数名 varchar(18) )
begin
    declare 变量名1 varchar(18);
    declare 变量名2 varchar(18);
    set 变量名1='值'; #赋值，无set为判断相等
    select 列名1,列名2 into 变量名1,变量名2 from 表名 where 列名='值' #从表中查询然后赋值给变量
    select 变量1; #输出

    if 条件式 then .......;
    elseif 条件式 then .......;
    else 条件式 then .......;
    end if;

    case 变量名
        when 值1 then ......;
        when 值2 then ......;
    end case;

    declare i int default 9
    repeat  #循环
        set num=num-1;
    untll num<1 end repeat;
    
    while i>0 do
        ......;
    end while
    
    标签名:while(true) do
        if 条件式 then iterate 标签名; #leave相当于continue
        if 条件式 then leave 标签名; #leave相当于break
    end while

end

call 函数名(); #执行

drop procedure 存储; #删除函数
```



### 函数

```mysql
select 列名1，if (列名2='值1','值1时显示值','非值1显示值') 称呼 from 表名; #根据不同值输出不同显示值
select 列名1，case 列名2 when '值1' then '显示值1' when '值2' then '显示值2' then ' ' from 表名; #根据不同值输出不同显示值
select ifnull('值1','值2') ; #值1为null则返回值2，否则返回值1
select nullif('值1','值2'); #值1=值2返回值1，否则返回null
select concate('值1','值2','值3'); #拼接字符串
concate_ws(sep,s1,s2...,sn); #将s1,s2...,sn连接成字符串，并用sep字符间隔
substring（被截取字段，从第几位开始截取，截取长度）
TRIM(str) #去除字符串首部和尾部的所有空格
UUID() #生成具有唯一性的字符串
LASTINSERTID() #返回最后插入的id值
CURDATE()或CURRENT_DATE() #返回当前的日期
CURTIME()或CURRENT_TIME() #返回当前的时间
DATE_ADD(date,INTERVAL int unit) #返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：SELECT DATE_ADD(CURRENT_DATE,INTERVAL 6 MONTH);
DATE_FORMAT(date,fmt) #依照指定的fmt格式格式化日期date值，如：select 
DATE_FORMAT(CURDATE(),'%Y-%m-%d')
DATE_SUB(date,INTERVAL int unit) #返回日期date减去间隔时间int的结果(int必须按照关键字进行格式化),如：SELECT DATE_SUB(CURRENT_DATE,INTERVAL 6 MONTH);
NOW() #返回当前的日期和时间
```

自定义函数

```mysql
CREATE FUNCTION sp_name ([param_name type[,...]])
RETURNS type -- 定义返回值类型
BEGIN
routine_body
END

#删除
DROP function [IF EXISTS] sp_name;
#查看
SHOW CREATE FUNCTION sp_name;
#使用
SELECT db_name.sp_name;
```



### 触发器

```mysql
#为表创建在 增/删/改 之前或之后 执行的触发器
create trigger 触发器名 before/after insert/delete/update 表名 for each ROW
BEGIN
    #触发器内容，例插入数据时检验输入性别是否正确，无效数据将性别设置为女
    if new.student_age<0 and new.student_age>150 then
        set new.student_age=old.student_age;
    end if;
END

insert into student VALUES('student013','小十三',13,'男男女女')
```



### 条件处理器

```mysql
declare [continue/exit/undo] handler for 条件
#条件
mysql_error_code | SQLSTATE [VALUE] sqlstate_value
| condition_name | SQLWARNING | NOT FOUND | SQLEXCEPTION
```



### 游标

```mysql
-- 根据课程号查询不及格名单(用游标)
create procedure pro (in couid)
begin
    declare stuid varchar(18);
    declare score varchar(18);
    declare flag int default false;
    declare cur cursor for select * from score where score<60 and course_id=couid; -- 定义游标
    declare continue handler for not found set flag=true; -- 定义处理器
    open cur; -- 打开游标
    w:while true DO
        if flag then
            leave w
        end if
        fetch cur into stuid,couid,score;
        select stuid,couid,score;
    end while w;
    close cur; -- 关闭游标
end
```

### 事务

```mysql
BEGIN #显式地开启一个事务；
START TRANSACTION #显式地开启一个事务；
COMMIT #也可以使用COMMIT WORK，不过二者是等价的。COMMIT会提交事务，并使已对数据库进行的所有修改称为永久性的；
ROLLBACK #有可以使用ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；
SAVEPOINT identifier #SAVEPOINT允许在事务中创建一个保存点，一个事务中可以有多个SAVEPOINT；
RELEASE SAVEPOINT identifier #删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；
ROLLBACK TO identifier #把事务回滚到标记点；
SET TRANSACTION #用来设置事务的隔离级别

#方式一：（用 BEGIN, ROLLBACK, COMMIT来实现）
BEGIN #开始一个事务
ROLLBACK #事务回滚
COMMIT #事务确认

#方式二：（直接用 SET 来改变 MySQL 的自劢提交模式）
SET AUTOCOMMIT=0 #禁止自劢提交
SET AUTOCOMMIT=1 #开吭自劢提交
```

### 备份

```mysql
#备份
mysqldump -u userName -p [arguments] > file_name.sql
#备份服务器上所有数据库
mysqldump -u userName -p --all-databases > file_name.sql
#备份指定数据库
mysqldump -u userName -p --databases > file_name.sql
#还原
mysql -u userName -p < file_name.sql
```





## 性能

### 指标

#### TPS

- TPS：TransactionsPerSecond（每秒传输的事物处理个数），这是指服务器每秒处理的事务数，支持事务的存储引擎如InnoDB等特有的一个性能指标。只适用于InnoDB，因为MyiISAM没有事务
- TPS = COM\_COMMIT + COM\_ROLLBACK) /UPTIME，（事务的提交次数+回滚次数）/mysql server运行时长，运行时长在mysql server重启后会刷新

```sql
use information_schema;
select VARIABLE_VALUE into @num_com from GLOBAL_STATUS where VARIABLE_NAME='COM_COMMIT';
select VARIABLE_VALUE into @num_roll from GLOBAL_STATUS where VARIABLE_NAME='COM_ROLLBACK' ;
select VARIABLE_VALUE into @uptime from GLOBAL_STATUS where VARIABLE_NAME='UPTIME';
select (@num_com+@num_ro11)/@uptime;
```

注：从mysql5.7.6开始information_schema.global_status已经开始被舍弃，为了兼容性，此时需要打开 show_compatibility_56。https://blog.csdn.net/itwxming/article/details/97897589

#### QPS

- QPS: Queries Per Second（每秒查询处理量）同时适用与InnoDB和MyISAM引擎
- QPS=QUESTIONS/UPTIME，每秒查询数量
- 等待时间：执行Sql等待返回结果之间的等待时间

```sql
use information_schema;
select VARIABLE_VALUE into @num_queries from GLOBAL_STATUS where VARIABLE_NAME='QUESTIONS';
select VARIABLE_VALUE into @num_roll from GLOBAL_STATUS where VARIABLE_NAME='COM_ROLLBACK' ;
select VARIABLE_VALUE into @uptime from GLOBAL_STATUS where VARIABLE_NAME='UPTIME';
select @num_queries/@uptime;
```



### 压测

#### MySqlSlap

- MySqlSlap：mysql提供的官方压测工具

- 安装：MySQLSlap是从MySQL的5.1. 4版开始就开始官方提供的压力测试工具。创建schema、table、 test data；运行负载测试，可以使用多个并发客户端连接；测试环境清理(删除创建的数据、表等，断开连接)

- 参数

  - --create-schema=name：指定测试的数据库名，默认是mysqlslap 
  - --engine=name：创建测试表所使用的存储引擎，可指定多个 
  - --concurrency=N：模拟N个客户端并发执行。可指定多个值，以逗号或者 
  - --number-of-queries=N：总的测试查询次数(并发客户数×每客户查询次数)，比如并发 是10，总次数是100，那么10个客户端各执行10个 
  - --iterations=N：迭代执行的次数，即重复的次数（相同的测试进行N次，求一 个平均值），指的是整个步骤的重复次数，包括准备数据、测 试load、清理 
  - --commit=N：执行N条DML后提交一次
  - --auto-generate-sql, -a：# 自动生成测试表和数据，表示用mysqlslap工具自己生成的 SQL脚本来测试并发压力。 
  - --auto-generate-sql-load-type=name：# 测试语句的类型。代表要测试的环境是读操作还是写操作还是两者混合的。# 取值包括：read (scan tables), write (insert into tables), key (read primary keys), update (update primary keys), or mixed (half inserts, half scanning selects). 默认值是：mixed. 
  - --auto-generate-sql-add- auto-increment：对生成的表自动添加auto_increment列 
  - --number-char-cols=name 自动生成的测试表中包含N个字符类型的列，默认1 
  - --number-int-cols=name：自动生成的测试表中包含N个数字类型的列，默认1 
  - --debug-info：打印内存和CPU的信息

- 案例：直接到bin目录中执行即可

  - 1000个客户端，重复10次，自动生成SQL语句，总共1000个查询：

    ./mysqlslap -uroot -proot --concurrency=1000 --iterations 10 -a --auto-generate-sql-add-autoincrement --engine=innodb --number-of-queries=1000

  - 1, 50，100， 200个客户端， 每一个测试3次， 并打印内存，CPU信息
    ./mysqlslap -uroot -proot --concurrency=1,50,100,200 --iterations=3 --number-char-cols=5 --number-int-cols=5 --auto-generate-sql --auto-generate-sql-add- autoincrement --engine=myisam,innodb --create -schema='enjoytest1' --debug-info

  - 500个客户端，分别使用myisam,innodb两种存储引擎，比较效率：

    ./mysqlslap -uroot -proot --concurrency=500 --iterations=3 --number-char-cols=5 --number-int-cols=5 --auto-generate-sql --auto-generate-sql-add-autoincrement --engine=myisam,innodb --create-schema='enjoytest1' --debug-info 



## 逻辑架构

这里是部分架构

### 连接层

- 当MySQL启动（MySQL服务器就是一个进程），等待客户端连接
- 每一个客户端连接请求，服务器都会新建一个线程处理（如果是线程池的话，则是分配一个空的线程），每个线程独立，拥有各自的内存处理空间，如果这个请求只是查询，没关系，但是若是修改数据，显然，当两个线程修改同一块内存是会引发数据同步问题的
- 连接到服务器，服务器寻需要对其进行验证，也就是IP、账户、密码验证，一旦连接成功，还要验证是否具有执行某个特定查询的权限（例如，是否允许客户端对某个数据库某个表的某个操作）

### SQL处理层

- 主要功能有：SQL语句的解析、优化；缓存的查询；MySQL内置函数的实现；跨存储引擎功能（所谓跨存储引擎就是说每个引擎都需提供的功能（引擎需对外提供接口），例如:存储过程、触发器、视图等。

- 如果是查询语句(select语句)，首先会查询缓存是否已有相应结果，有则返回结果，无则进行下一步（如果不是查询语句，同样调到下一步）

- 解析查询，创建一个内部数据结构（解析树），这个解析树主要用来SQL语句的语义与语法解析;

- 优化：优化SQL语句，例如重写查询，决定表的读取顺序，以及选择需要的索引等。这一阶段用户是可以查询的，查询服务器优化器是如何进行优化的，便于用户重构查询和修改相关配置，达到最优化。这一阶段还涉及到存储引擎，优化器会询问存储引擎，比如某个操作的开销信息、是否对特定索引有查询优化等。

- 缓存：如果缓存中有就执行缓存中的计划（如果开启了数据缓存则直接返回结果）
  - 缓存sql（及其执行计划），默认开启
  - 缓存数据，默认不开启。
    - 数据缓存是否开启的参数：show variables like '%query_cache_type%'。一般不开启，而是用redis等专业的缓存
    - 数据缓存大小：show variables like '%query_cache_size%'，修改大小SET GLOBAL query_cache_size=11111111;或者在mysql的配置文件my.ini中修改
  
- 查询解析器：解析成mysql识别的东西，见图7

  - FROM：笛卡尔积
  - ON：主表保留
  - JOIN：不符合ON也添加。WHERE：非聚合；非SELECT别名
  - GROUP BY：改变对表引用
  - HAVING：只作用分组后
  - SELECT：DISTINCT
  - ORDER BY：可使用SELECT别名
  - LIMIT：rows；offset

- 查询优化器：explan查看优化结果，即执行计划。例如执行下面两个sql可以发现它们执行计划是完全一样的，where 1=1被查询优化器优化排除了

  ```sql
  explain select * from user where 1=1
  explain select * from user
  ```

  

### 存储引擎

- 查看：通过 `show engines` 查看看当前 mysql 提供的存储引擎；通过 `show variables like '%storage_engine%'` 查看 mysql 当前默认存储引擎
- 存储引擎的索引
  - 聚集索引：索引项顺序 与 数据存储顺序 一致
    - 即聚集索引的顺序就是数据的物理存储顺序。它会根据聚集索引键的顺序来存储表中的数据，即对表的数据按索引键的顺序进行排序，然后重新存储到磁盘上。因为数据在物理存放时只能有一种排列方式，所以一个表只能有一个聚集索引
    - 场景：查询命令的回传结果是以该字段为排序依据的；查询的结果返回一个区间的值；查询的结果返回某值相同的大量结果集。这都得益于索引排序与数据排序一致
    - 优：select适用性高。从场景就可以发先聚集索引的查询很符合我们最普遍的逻辑。
    - 劣：聚集索引会降低 insert，和update操作的性能，所以，是否使用聚集索引要全面衡量
  - 非聚集索引：索引项顺序 与 数据存储顺序 一致
    - 非聚集索引必须是稠密索引
    - 场景：查询所获数据量较少时；某字段中的数据的唯一性比较高时；
  - 聚簇索引：聚簇索引并不是一种单独的索引类型，而是一种数据存储方式。术语“聚族”表示数据行和相邻的键值紧凑的存储在一起。因为无法同时把数据行放在两个不同的地方，所以一个表只能有一个聚族索引。
    - 优点：可以把相关数据保存在一起。就好像在操场上战队，一个院系一个院系的站在一起，这样要找到一个人，就先找到他的院系，然后在他的院系里找到他就行了，而不是把学校里的所有人都遍历一遍。数据访问更快。聚族索引将索引和数据保存在同一个B-Tree中，因此从聚族索引中获取数据通常比在非聚族索引中查找更快
  - 稠密索引：每个索引键值都对应有一个索引项，索引项指向的索引键值更连续
    - 稠密索引能够比稀疏索引更快的定位一条记录
  - 稀疏索引：每个索引键值都对应有一个索引项，索引项指向的索引键值更分散
    - 相对于稠密索引，稀疏索引只为某些搜索码值建立索引记录；在搜索时，找到其最大的搜索码值小于或等于所查找记录的搜索码值的索引项，然后从该记录开始向后顺序查询直到找到为止。



#### InnoDB

- Innodb：mysql5.5 及之后版本默认使用的存储引擎。

- **存储**：**frm 文件存储表结构，ibd 文件存储索引+数据**。可见索引与数据是存储在一个文件中的

- **索引**：**聚集索引**，索引项顺序 与 数据存储顺序 一致

  - 也使用 B+Tree 作为索引结构，索引页大小16，和表数据页共同存放在表空间中，可以看出InnoDB表数据文件本身就是按 B+Tree 组织的一个索引结构，这棵树的叶节点 data 域保存了完整的数据记录。这个索引的key是数据表的主键，因此 InnoDB 表数据文件本身就是主索引。
  - InnoDB默认对主键建立聚簇索引。如果你不指定主键，InnoDB会用一个具有唯一且非空值的索引来代替。如果不存在这样的索引，InnoDB会定义一个隐藏的主键，然后对其建立聚簇索引。一般来说，InnoDB 会以聚簇索引的形式来存储实际的数据，它是其它二级索引的基础。
  - 所以mysql innodb引擎的聚集索引、聚簇索引都默认是主键索引，如果没有指定主键，就是一个具有唯一且非空值的索引，如果不存在这样的索引，就是InnoDB自定义的隐藏主键索引，并且该索引是稠密索引。
  - 支持主外键

- **锁**：默认采用行级锁，并发能力稍好；当然表级锁也是支持的

- **表空间**：通过 `innodb_ file_ per_ table` 配置 innodb 的表是否使用单独数据文件。配置为 ON 时开启 独立表空间；配置为 OFF 时则是 系统表空间。

  - **独立表空间**：每个表使用自己单独的文件。会在对应 schame 文件夹下生成 frm 和 idb 文件，以表名为名，table_name.frm、table_name.ibd（索引+数据）。通过 `optimize table table_name` 可以简便的收缩系统文件，独立表空间可以同时向多个文件刷新数据。MySQL5.6 及之后默认为独立表空间。

    ```sql
    -- 删除大量数据，再执行OPTIMIZE TABLE table_name，可以发现.idb文件大小变小了很多，类似于磁盘整理，这就是收缩系统文件
    DELETE user WHERE id!=1
    OPTIMIZE TABLE user
    ```

  - **系统表空间**：每个表没有单独的文件，所有 innodb 的表共用一个数据文件。schame 文件夹下仍然会生成其 frm 文件，table_name.frm，但是索引和数据将存放在data/目录下的idata文件中，是一个系统级的数据文件。无法简单的收缩文件大小，随着数据量增大，系统表空间会存在 IO 瓶颈。mysql5.6 之前默认为系统表空间

  - 建议：Innodb 建议使用 独立表空间

- **事务**：Innodb 是一种事务性存储引擎，完全支持事务得 ACID 特性，Redo Log 和 Undo Log

- **缓存**：缓存索引+数据。对内存要求较高，而且内存大小对性能有决定性的影响

- 其它：支持全文检索（mysql5.6 及之后）；支持数据压缩，如前面 optimize 操作；

- 适用场景：Innodb 适合于大多数 OLTP（on-line transaction processing，联机事务处理）应用



#### MyISAM

- MyISAM：mysql5.5 之前默认使用的存储引擎
- **存储**：**frm 文件存储表结构，MYI 文件存储索引，MYD 文件存储数据**。可见索引与数据是分开存储的
- **索引**：**非聚集索引**，索引项顺序 与 数据存储顺序 不一致
  - 索引文件仅保存数据记录的地址，主索引和辅助索引无区别
  - 索引页正常大小为1024字节，索引页存放在.MYI 文件中
  - 使用 B+Tree 作为索引结构，叶节点的 data 域存放的是数据记录的地址
  - 不支持主外键
- **锁**：只支持表级锁，并发能力自然一般
- **事务**：不支持
- **缓存**：只缓存索引，不缓存数据
- 其它
  - 支持全文检索
  - 支持数据压缩：由 bin/myisampack.exe 程序支持，通过命令 `myisampack -b -f table_name.MYI` 可以压缩数据文件，压缩后旧文件会变成 .OLD 文件。注意如果删除 .OLD 文件，查询可以正常查询，但是增删改可能会出问题，通过 `CHECK table product info` 可以检查是否存在问题，通过 `REPAIR table product info` 可以恢复原文件。
- 适用场景：**临时表**、非事务型应用（如数据仓库、日志数据、报表等）；只读类应用，查询快；空间类应用(空间函数，坐标)，详见文档mysl5空间扩展。



#### Memory

- Memory：也称 HEAP 存储引擎，数据保存在内存中，所有连接有效，重启mysql就没了
- **索引**：支持 HASH 索引和 BTree 索引
- **锁**：表级锁
- 其它：所有字段都是固定长度varchar(10) = char(10)；不支持Blog和Text等大字段；最大大小由max_ heap_ table_size参数决定
- 使用场景：
  - **临时表**：保存一些临时数据
    - 临时表只在一个连接中有效
    - 临时表都是由 mysql 系统自己创建和维护的，通过 `create temporary table table_name{...}` 可以创建临时表
    - mysql 自己就是使用 Memory 或 Myisam 存储引擎来实现临时表。临时表大小不超过配置限制，则使用 Memory 表；超过限制将使用 Myisam 表。Memory 表存储在内存，Myisam 表存储在磁盘，可见临时表过大将严重影响性能
  - hash 索引用于查找或者是映射表（邮编和地区的对应表）
  - 用于保存数据分析中产生的中间表
  - 用于缓存周期性聚合数据的结果表
  - 数据可再生。memory数据易丢失，所以要求数据可再生



#### CSV

- 存储：frm 文件存储表结构，csm 文件存储表得元数据如表状态和数据量，csv 文件存储数据。数据以文本的方式存储
- 特点：以csv格式进行数据存储；所有列都不能为null；不支持索引(不适合大表，不适合在线处理)；可以对数据文件直接编辑(保存文本文件内容，每一行要以回车\n结尾)：需要flush tables; 刷新表



#### Archive

- 存储：frm 文件存储表结构，ARZ 文件存储数据。是以 zlib 对表数据进行了压缩，磁盘I/O更少
- 其它特性：只支持insert和select操作，只允许在自增ID列上加索引
- 使用场景：日志和数据采集应用（当然一般如果用到了 mongodb 就使用 mongodb 即可）



#### Ferderated

- 其它：提供了访问远程MySQL服务器上表的方法；本地不存储数据，数据全部放到远程服务器上；本地需要保存表结构和远程服务器的连接信息
- 适用场景：偶尔的统计分析及手工查询
- 开启：通过 `show engine` 查看是否开启，默认不开启，需要手动开启，在配置文件 my.ini 中加上配置 `ferderated=1` 也可以开启

```sql
--local_fed要与下面remote_fed表结构一致
CREATE TABLE local_fed (
`id` int(11) NOT NULL AUTO_ INCREMENT,
`c1` varchar (10) NOT NULL DEFAULT '',
`c2` char(10) NOT NULL DEFAULT '',
PRIMARY KEY (`id`)
) ENGINE=federated CONNECTION ='mysq1:/ / root:root@127.0.0.1:3306/remote/remote_fed'
```


## 锁

- 行级锁：

  - 优劣：开销大，加锁慢；锁定粒度最小，发生锁冲突的概率最低，并发度也最高，但会出现死锁。
  - 适用：适合于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用。如一些在线 OLTP（on-line transaction processing，联机事务处理）系统。

- 表级锁：

  - 优劣：开销小，加锁快；锁定粒度大，发生锁冲突的概率最高，并发度最低，但不会出现死锁
  - 适用：适合于以查询为主，只有少量按索引条件更新数据的应用。如OLAP（On-Line Analytical Processing，联机分析处理）系统

- 页面锁

  - 优劣：开销和加锁时间界于表锁和行锁之间:会出现死锁:锁定粒度界于表锁和行锁之间，并发度一般。

- 仅从锁的角度来说（很难笼统地说哪种锁更好，只能就具体应用的特点来说哪种锁更合适）

  



### 表锁

- 表锁：就是对表的读写锁

  - Table Read Lock：表读锁。一个 session 对一个 table 的读操作会导致 表读锁。因为是存在读共享的，所以也称为表共享读锁

    - 其它session操作被锁table：**读无阻塞共享，写会等待阻塞**

    - 上锁session操作被锁table：读无阻塞，写err

    - 上锁session操作其它table：上锁session被锁在表中，对其它表执行任何读写操作都将err（注意：被锁表取别名也将被识别为其它表）

    - 通过 `lock table 表名 read` 可以手动上表读锁
    
      ```sql
      lock table user read; --上表共享读锁
      update user set id=2 where id=1; --报错
      update user_info set id=2 where id=1; --对另一张表执行写操作也报错，是session写 被关在表中了
      select * user; --成功，读无阻塞共享
      select u.* user fom user u; --如果是别名也会报错，a会被识别为非user表
  lock table user as u Read; --如果非要用别名，那也必须要用同名别名来上锁
      ```

      ```sql
    --另一个session。user被读锁锁住期间，在另一个session中执行写操作，不会报错但是会等待，直到user释放锁则成功，否则阻塞时间过长将超时
      update user set id=2 where id=1; --等待
      ```

  - Table Write Lock：表写锁，表独占写锁。一个session对一个table的写操作会导致 表独占写锁（手动上锁：lock table 表名 write）。写锁，即对表的写操作导致的锁，因为读写都是独占的，所以被称为独占

    - 其它session操作被锁table：读写均等待阻塞
    - 上锁session操作被锁table：**独占**，读写均可成功
    - 上锁session操作其它table：上锁 session 被锁在表中，对其它表执行任何读写操作都将err（注意：被锁表取别名也将被识别为其它表）
    - 通过 `lock table 表名 write` 可以手动上表写锁
  
- 锁表因素：

  - 非索引查询会导致表锁，但 MySQL 做了优化，对于不满足条件的记录，会在判断后**释放锁**，最终持有的，是满足条件的记录上的锁
  - 表结构修改会导致表锁



### 行锁

- MySQL的InnoDB引擎支持行锁和表锁。数据库使用锁是为了支持更好的并发，提供数据的完整性和一致性。InnoDB是一个支持行锁的存储引擎，锁的类型有：共享锁（S）、排他锁（X）、意向共享（IS）、意向排他（IX）。为了提供更好的并发，InnoDB提供了非锁定读：不需要等待访问行上的锁释放，读取行的一个快照。该方法是通过InnoDB的一个特性：MVCC来实现的。

- 行锁：不一定是一行，可能是多行。如页锁 where id>3 and id<9；如间隙锁 where id=3 or id=9。提交或回滚事务可以释放锁。

  - **共享锁**：又称读锁。多个事务的查询语句可以共用某数据的共享锁，**读无阻塞共享**

    - 如果**只有一个事务拿到了共享锁**，则该事务可以对数据进行 UPDATE DETELE 等写操作；
    - 如果**有多个事务拿到了共享锁**，则所有事务都不能对数据进行 UPDATE DETELE 等写操作。

    ```sql
    --关闭自动提交
    begin --开启事务
    select * from user where id=1 lock in share mode; --上共享锁
    commit; --提交或回滚事务可以释放锁
    ```

  - **排它锁**：又称写锁。只有一个事务能获取某数据的排它锁，该事务可以读写该数据，但其它事务对该数据**读写等待阻塞**。

    ```sql
    select from user where id=1 for update; --上排它锁
    ```

- 意向锁：意向锁的存在是为了协调行锁和表锁的关系，支持多粒度（表锁与行锁）的锁并存

  - **意向共享锁**：IS锁。一个事务给一个数据行加共享锁时，必须先获得表的IS锁。如果事务B要给表上一个表独占锁，判断意向锁就知道表中已经有在使用共享锁，就会阻塞
  - **意向排他锁**：IX锁。一个事务给一个数据行加排他锁时，必须先获得表的IX锁。如果事务B要给表上一个表锁，判断意向锁就知道表中已经有在使用排他锁了，就会阻塞

- 行锁的实现算法：https://www.oschina.net/question/4154815_2315824，https://www.cnblogs.com/zhoujinyi/p/3435982.html

  - **Record Lock**：记录锁，**单个行记录上的锁**。

    - Record Lock总是会去锁住索引记录，如果InnoDB存储引擎表建立的时候没有设置任何一个索引，这时InnoDB存储引擎会使用隐式的主键来进行锁定

  - **Gap Lock**：间隙锁，**锁定一个范围，但不包含记录本身**，锁定已有记录之间的还未存在的值范围，或者第一条索引记录之前的范围，又或者最后一条索引记录之后的范围

    - 产生间隙锁的条件（RR事务隔离级别下）：使用普通索引（非唯一索引）锁定；使用多列唯一索引；使用唯一索引锁定多行记录。使用单个唯一索引不会产生间隙锁，比如id=5，只会有记录锁

    - 可以防止同一事务的两次当前读，解决幻读问题

    - 通过 `show variables like 'innodb_locks_unsafe_for_binlog'` 查看是否禁用间隙锁，该参数是只读模式，通过配置文件 my.ini 配置可以禁用

      ```conf
      [mysqld]
      innodb_locks_unsafe_for_binlog = 1
      ```

  - **Next-Key Lock**：临键锁=行锁+间隙锁，锁定一个范围，并且锁定记录本身。这个锁算法实际上就是对行锁和间隙锁的灵活应用，有时候只使用行锁，有时候只使用间隙锁，有时候同时一起使用

    - 在Repeatable Read隔离级别下，Next-key Lock 算法是默认的行记录锁定算法。如果把事务的隔离级别降级为RC，临键锁则也会失效
    - 范围是前开后闭的 ( ] 。比如存在记录 id=3,5 ，那么可能被锁定的范围就是(-, 3]、(3, 5]、(5, +]，此时查询 id=5 时，即锁定 (3, 5]。
    - 当查询的索引有唯一属性时，InnoDB 会对 Next-Key Lock 进行优化，将其降为 Record Lock，即仅锁住索引本身，而不是范围；但是如果唯一索引由多个列组成，而查询仅是查找多个列中的一个，那么查询其实是range 类型查询，而不是 point 类型查询，故 InnoDB 仍然使用 Next-Key Lock 进行锁定
    - 聚集索引和辅助索引同时存在时，当sql语句通过辅助索引列进行查询，会对两个索引分别进行锁定，满足辅助索引查询条件的行中的聚集索引使用 Record Lock 锁定，辅助索引使用 Next-Key Lock 锁定，且 InnoDB还会对辅助索引的下一个相等键值加上 Gap Lock，这样才能避免幻读。因为辅助索引可以是不唯一的，即使锁住了已有记录，其它事务仍然可以插入同值的记录，导致幻读
      - 比如一个表 `CREATE TABLE z (a INT,b INT,PRIMARY KEY(a), KEY(b)); INSERT INTO z SELECT 3,2; INSERT INTO z SELECT 5,4; INSERT INTO z SELECT 7,6;` ，唯一索引a，辅助索引b
      - 当使用辅助索引 b 查询时 `WHERE b=4` ，记录为 `5,4`，对于唯一索引 a 将使用 Record Lock 锁定 a=5，对于辅助索引 b 将使用 Next-Key Lock 锁定 b=(2,4] ，并且s使用 Gap Lock 锁定 b=(4,6)。因为 InnoDB 的 insert 操作会检查插入记录的下一条记录是否被锁定，如果被锁定则不允许查询，自然也无法执行后续的插入了。所以如果不加这个 Gap Lock，即使已经锁定了已有的 b=4 的记录，b=5 并没有被锁定，其它事务仍然可以而插入一个 b=4 的记录，这将导致条件 `where b=4` 在同一事务中的再次查询出现幻读

  

- 注意：两个事务不能锁同一个索引。insert、delete、update操作在事务中都会自动默认加上排它锁。

- 面试题：系统运行一段时间，数据量已经很大，这时候系统升级，有张表A需要增加个字段，并发量白天晚上都很大，请问怎么修改表结构。面试考点：修改表结构会导致表锁,数据量大修改数据很长，导致大量用户阻塞，无法访问! 

  1. 首先创建一个和你要执行的alter操作的表一样的空的表结构
  2. 执行我们赋予的表结构的修改，然后copy原表中的数据到新表里面。
  3. 在原表上创建一个触发器，在数据copy的过程中，将原表的更新数据的操作全部更新到新的表中来。
  4. copy完成之后，用rename table新表代替原表，默认删除原表。

- 物理结构修改-pt-online-schema-change，一个表结构修改工具

  - 下载安装 perl 环境：http://www.perl.org/get.html
  - 下载 percona-toolkit 工具集合：https://www.percona.com/doc/percona-tookit
  - ppm install DBI：依赖
  - ppm install DBD::mysql：安装mysq|驱动依赖
  - 修改表结构（其实就是完成了上面面试题答案的步骤）：pt-online-schema-change h=127.0.0.1,u=root,D=database_name,t=table_name --alter "modify colunm_name varchar(150) not null default ' ' " -execute



## 事务

现在的很多软件都是多用户，多程序，多线程的，对同一个表可能同时有很多人在用，为保持
数据的一致性，所以提出了事务的概念。
A给B要划钱，A的账户-1000元，B 的账户就要+1000元，这两个update语句必须作为一一个整体
来执行，不然A扣钱了，B没有加钱这种情况很难处理。

show engines，mysql中只有只有innodb存在事务

1.查看数据库下面是否支持事务( InnoDB支持) ?
show engines;
2.查看mysql当前默认的存储引擎?
show variables like '%storage_ _engine%';
3.查看某张表的存储引擎?
show create table表名;
4.对于表的存储结构的修改?
建立InnoDB表: Create table ... type=InnoDB; Alter table table_ name type=InnoDB;

事务的特性
事务应该具有4个属性:原子性、- -致性、隔离性、持久性。这四个属性通常称为ACID特性。
◆原子性(atomicity) 。-一个事务是一个不可分割的工作单位(最小单元)，事务中包括的诸操作要么都做，要么都不做。一个人转给另一个人100块，A-100和B+100是一个一个整体必须都发生
◆一致性(consistency)。事务必须是使数据库从一个--致性状态变到另一个--致性状态。一致性与原子性是密切相关的。A-100，B必须只能+100，不能出现如B+100后系统卡顿，没有得到响应，又让B+100。说简单一点就是要与事务所期望的数据结果完全一致，主要体现在不能让其它事务干扰修改数据导致数据错误
◆隔离性(isolation) 。一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。
◆持久性(durability) 。持久性也称永久性( permanence)，指一个事务- -旦提交，它对数据库中数据的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响。持久性不能完全通过数据库本身解决，需要做如主从、容灾等（几地几中心）



- 事务并发问题
  - **脏读**：一个事务读取到了其它事务修改前的数据。问题源于其它事务对数据的并发修改
  - **不可重复读**：一次事务中重复读取同一数据，多次读取之间数据不一致，违背了事务一致性状态，因此不可重复读。问题源于其它事务对数据的并发修改。行锁可以解决问题
  - **幻读**：一个事务中年重复读取一些数据，非第一次读取时读到了之前没有返回的记录，如幻影一般。问题源于其它事务对数据的并发增删。几个解决思路如下
    - next key lock：等于record Lock + gap Lock，锁定记录本身并锁定一个范围，可以完美控制**当前读**的幻读问题
    - 表锁：简单粗暴
    - MVCC：https://blog.csdn.net/w2064004678/article/details/83012387。Multi-Version Concurrency Control，多版本并发控制，类似于乐观锁的一种实现方式。通过保存数据在某个时间点的快照，以**快照读**的方式读取数据，快照读从根本上就不存在幻读问题，但并不能阻止其它事务对表进行写操作，等于自欺欺人
  
- 事务隔离级别（重点）：事务隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大，对于多数应用程序，可以优先考虑把数据库系统的隔离级别设为ReadCommitted,它能够避免脏读取，而且具有较好的并发性能。

  ```sql
  show variables like '%tx_isolation%'; --查看当前事务隔离级别mysq|默认的事务隔离级别为repeatable-read
  --修改全局隔离级别可以通过配置文件
  set SESSION TRANSACTION ISOLATION LEVEL   READ UNCOMMITTED; --修改当前session隔离级别
  ```

  - READ UNCOMMITED：**读未提交**。一个事务可以读取其它事务未提交的数据。存在问题：脏读、不可重复度、幻读
    - 读未提交：如事务B修改了一行数据，还未提交，随之事务A成功读取到B未提交的修改过的数据，即读未提交
    - 脏读：但是A读取到B修改过的数据之后，B回滚了数据，A读到的数据与数据库真实数据不符，该数据被称为脏数据，产生脏读
  - READ COMMITED：**读已提交**。一个事务可以读取其它事务已经提交的数据。存在问题：不可重复读、幻读。
    - 读已提交：如A事务将对同一行数据执行2次读取，但是A读了一次之后，事务B修改了该行数据并完成提交，那么事务A第二次读取到的数据是事务B已提交的新数据，即可以读取其它事务已经提交的数据
    - 不可重复读：因为上面两次读取到的数据不一致，不满足事务的一致性状态，即不可重复读
    - 在 READ COMMITED 下，除了唯一性的约束检查及外键约束检查需要 Gap Lock，InnoDB 不会使用 Gap Lock    锁算法
    - READ COMMITED 下，通过 MVCC 解决了不可重复读。如果事务执行时间较短，其实没太多影响，因为本来就是以事务开始的时间作为节点去查询这个节点上的数据；但是如果执行时间特别长，甚至是一些失误写出的超级慢慢sql，那就可能会一定程度破坏数据一致性，就有些本质上治标不治本了
  - REPEATABLE READ：**可重复读**。存在问题：幻读。
    - 可重复度：如事务A将对同一行数据执行2次读取，但是A读了一次之后，事务B修改了该行数据并完成提交，事务A第二次读取到的数据仍然将与第一次的数据一致，而读取不到B提交的新数据，满足了事务的一致性状态，即可重复读；
    - 幻读：会话A开启事务A→会话B开启事务B→事务A修改数据,并查询到2条数据(A的读取到的第二条数据是A添加的数据)→事务B添加一条数据,并查询到2条数据(B的读取到的第二条数据是B添加的数据)→事务A提交→事务B提交→会话A再次查询数据发现有3条记录，就像幻觉→会话B再次查询数据发现有3条记录，就像幻觉。A和B均产生幻读
    - 产生锁：事务隔离级别为可重复读时
      - 行锁&页锁：如果where涉及索引（包括主键索引），以索引列为条件更新数据，会存在间隙锁间、行锁、页锁的问题，从而锁住一些行
      - 表锁：如果where条件不涉及索引，更新数据时会锁住整张表
      - **行锁升级表锁**：索引失效
  - **SERIALIZABLE**：序列化，串行化。事务序列化串行执行。不存在任何问题
    - 锁
      - 表锁：事务隔离级别为串行化时，读写数据都会锁住整张表

- 事务操作语法

  - 事务自动提交关闭

    ```sql
    set autocommit=0 --或者修改配置文件my.ini
    ```

  - 事务开启

    ```sql
    begin
    begin work
    start transaction --推荐
    ```

  - 事务回滚

    ```sql
    rollback --回滚整个事务
    rollback to savepoint sp_name --回滚到某个还原点
    ```

  - 事务提交

    ```sql
    conmmit
    ```

  - 事务还原点设置：还原到某个点

    ```sql
    insert into user values(1,1,1) --添加第1条记录
    savepoint sp1                  --设置还原点sp1
    insert into user values(2,2,2) --添加第2条记录
    savepoint sp2                  --设置还原点sp2
    insert into user values(3,3,3) --添加第2条记录
    savepoint sp3                  --设置还原点sp3
    select * from user             --表中有3条记录
    rollback to savepoint sp1      --回滚到还原点sp1
    select * from user             --表中有1条记录
    ```



## 业务设计

### 逻辑设计

##### 范式设计

- 数据库设计三大范式：除了第一大范式，其它每个范式都是建立在前一个范式基础之上的

  - 第一范式（1NF）：列的原子性，最小属性不可再分化。数据表的每一列，必须是不可拆分的最小单元

    - 错误示范

      - user

        | id   | name-age  |
        | ---- | --------- |
        | 1    | yuanya-23 |

    - 正确示范

      - user

        | id   | name   | age  |
        | ---- | ------ | ---- |
        | 1    | yuanya | 23   |

  - 第二范式（2NF）：满足1NF的前提下。行的唯一性，非主键列完全依赖主键列，不是部分依赖关键字

    - 错误示范：一个订单有多个产品

      - order

        | id   | product_id | time       |
        | ---- | ---------- | ---------- |
        | 1    | 1          | 2020-02-02 |
        | 1    | 2          | 2020-02-02 |

    - 正确示范

      - order

        | id   | time       |
        | ---- | ---------- |
        | 1    | 2020-02-02 |

      - order_product

        | id   | order_id | product_id |
        | ---- | -------- | ---------- |
        | 1    | 1        | 1          |
        | 2    | 1        | 2          |

  - 第三大范式：满足1NF的前提下。非主键列直接依赖主键列，不是间接依赖。数据库表中不包含已在其它表中已包含的非主关键字信息

    - 错误示范

      - order_product（这非常不利于增改，增改都要多考虑一个列，但是其实有时候为了查询方便会设计这样的冗余字段，这是一种反范式设计）

        | id   | order_id | product_id | product_name |
        | ---- | -------- | ---------- | ------------ |
        | 2    | 1        | 1          | 飞机         |
        | 2    | 1        | 2          | 大炮         |

- 例子

  - 用户表：用户名  密码  手机号  姓名  注册时间  在线状态  出生日期。符合三大范式
  - 商品表：ID  商品名称  分类名称  出版社名称  图书价格  图书表述  作者。不符合，一个图书通常有多种分类，应该设计成多对多的形式
  - 出版社表：出版社名称  地址  电话  联系人  银行账号。符合
  - 出版社表：出版社名称  地址  电话  联系人  银行账号  银行支行。不符合，因为 银行账号、银行支行 之间存在关联关系
  - 订单表：订单编号(pK)  下单用户名  下单日期  订单金额  订单商品分类  订单商品名(pk)  订单商品单价  订单商品数量  支付金额  物流单号。有多个业务主键，不符合第二范式；订单商品单价、订单数量、订单金额 存在传递依赖关系，不符合第三范式。这样计算得到的订单金额会随着商家升降价改变，这是不被允许的，所以可见完全遵循三大范式有时候并不可行

- 订单

  - 表设计

    - 订单表：订单编号、下单用户名、下单日期、支付金额、物流单号
    - 订单商品关联表：订单编号、商品分类ID、订单商品数量
    - 商品信息表：商品名称、出版社名称、图书价格、图书表述、作者
    - 商品分类关联表：商品分类ID、商品名称、分类名称
    - 商品分类信息表：分类名称、分类描述
    - 供应商信息表：出版社名称、地址、电话、联系人、银行账号
    - 用户表：用户名、密码、手机号、姓名、注册时间、在线状态、出生日期

  - sql

    - 编写SQL查询出每一个用户的订单总金额

      ```sql
      SELECT a.单用户名, sum(d.商品价格 * b.商品数量)
      FROM 订单表 a
      JOIN 订单分类关联表 b ON a.订单编号 = b.订单编号
      JOIN 商品分类关联表 c ON c.商品分类ID = b.商品分类ID
      JOIN 商品信息表 d ON d.商品名称 = c.商品名称
      GROUP BY a.下单用户名
      ```

    - 编写SQL查询出下单用户和订单详情

      ```sql
      SELECT a.订单编号, e.用户名, e.手机号, d.商品名称, c.商品数量, d.商品价格
      FROM 订单表 a
      JOIN 订单分类关联表 b ON a.订单编号 = b.订单编号
      JOIN 商品分类关联表 c ON c.商品分类ID = b.商品分类ID
      JOIN 商品信息表 d ON d.商品名称 = c.商品名称
      JOIN 用户信息表 e ON e.用户名 = a.下单用户
      ```

  - 问题：完全符合范式化的设计有时并不能得到良好得SQL查询性能，大量的表关联非常影响查询的性能

- 优点：

  - 可以尽量得减少数据冗余
  - 范式化的更新操作比反范式化更快
  - 范式化的表通常比反范式化的表更小

- 缺点：

  - 对于查询需要对多个表进行关联

  - 更难进行索引优化 

##### 反范式设计

- 反范式设计：所谓得反范式化就是为了性能和读取效率得考虑而适当得对数据库设计范式得要求进行违反。允许存在少量得冗余，换句话来说反范式化就是使用空间来换取时间。主要是将一些经常多表关联查询的字段在被查表中做冗余

- 表设计：

  - 商品

    - ID  商品名称  分类名称  出版社名称  图书价格  图书表述  作者
    - 分类信息：商品名称、分类名称

  - 订单

    - 订单表：订单编号  下单用户名  下单日期  支付金额  物流单号
    - 订单商品关联表：订单编号  订单商品数量  订单商品名  订单商品分类  订单单价

  - sql

    - 编写SQL查询出每一个用户的订单总金额

      ```sql
      SELECT 下单用户名, sum(订单金额)
      FROM 订单表
      GROUP BY 下单用户名;
      ```

    - 编写SQL查询出下单用户和订单详情

      ```sql
      SELECT a.订单编号, a.用户名, a.手机号, b.商品名称, b.商品单价, b.商品数量
      FROM 订单表 a
      JOIN 订单商品关联表 b ON a.订单编号 = b.订单编号
      ```

- 优点：

  - 可以减少表的关联
  - 可以更好的进行索引优化

- 缺点

  - 存在数据冗余及数据维护异常
  - 对数据的修改需要更多的成本

  

### 物理设计

- 物理设计
  - 定义数据库、表及字段的命名规范
  - 选择合适的存储引擎
  - 为表中的字段选择合适的数据类型
  - 建立数据库结构

##### 命名规范

- 数据库、表、字段的命名要遵守可读性原则
  使用大小写来格式化的库对象名字以获得良好的可读性
  例如：使用custAddress而不是custaddress来提高可读性。
- 数据库、表、字段的命名要遵守表意性原则
  对象的名字应该能够描述它所表示的对象
  例如：
  对于表，表的名称应该能够体现表中存储的数据内容；对于存储过程
  存储过程应该能够体现存储过程的功能。

- 数据库、表、字段的命名要遵守长名原则
  尽可能少使用或者不使用缩写

##### 存储引擎选择

| **对比项** | **MyISAM**                                             | **InnoDB**                                                   |
| ---------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| 主外键     | 不支持                                                 | 支持                                                         |
| 事务       | 不支持                                                 | 支持                                                         |
| 行表锁     | 表锁，即使操作一条记录也会锁住整个表不适合高并发的操作 | 行锁,操作时只锁某一行，不对其它行有影响适合高并发的操作      |
| 缓存       | 只缓存索引，不缓存真实数据                             | 不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响 |
| 表空间     | 小                                                     | 大                                                           |
| 关注点     | 性能                                                   | 事务                                                         |
| 默认安装   | Y                                                      | Y                                                            |

##### 数据类型选择

- 当一个列可以选择多种数据类型时

  - 优先考虑数字类型
  - 其次是日期、时间类型
  - 最后是字符类型
  - 对于相同级别的数据类型，应该优先选择占用空间小的数据类型

- 浮点类型

  | **列类型** | **存储空间**                        | **是否精确类型**       |
  | ---------- | ----------------------------------- | ---------------------- |
  | FlOAT      | 4个字节                             | 否，存在精度丢失       |
  | DOUBLE     | 8个字节                             | 否，存在精度丢失       |
  | DECIMAL    | 每4个字节存9个数字，小数点占1个字节 | 是，如财务数据优先考虑 |

- 时间类型

  | 类型      | 大小（字节） | 范围                                              | 格式                | 用途                     | 时区                     |
  | --------- | ------------ | ------------------------------------------------- | ------------------- | ------------------------ | ------------------------ |
  | DATETIME  | 8            | 1000-01-01 00:00:00 ~ 9999-12-31 23:59:59         | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         | 时区无关，修改时区无改变 |
  | TIMESTAMP | 4            | 1970-01-01 00:00:01 UTC ~ 2038-01-19 03:14:07 UTC | YYYY-MM-DD HH:MM:SS | 混合日期和时间值，时间戳 | 时区有关，修改时区会改变 |
  | DATE      | 3            | 1000-01-01 ~ 9999-12-31                           | YYYY-MM-DD          |                          |                          |
  | TIME      | 3            | -838:59:59 ~ 838:59:59                            | HH:MM:SS            |                          |                          |
  | YEAR      | 1            | 1901 ~ 2155                                       | YYYY-               |                          |                          |

  

## 慢查询

慢查询定义及作用
慢查询日志，顾名思义，就是查询慢的日志，是指mysql记录所有执行超过long_query_time参数设定的时间阈值的SQL语句的日志。该日志能为SQL语句的优化带来很好的帮助。默认情况下，慢查询日志是关闭的，要使用慢查询日志功能，首先要开启慢查询日志功能。

### 配置

```sql
set global slow_query_log=1 --单位秒,超过1秒的操作将被记录日志
show variable like '%slow_query_log%'
```

- 常用配置

  - slow_query_log 启动停止技术慢查询日志
  - slow_query_log_file 指定慢查询日志得存储路径及文件（默认和数据文件放一起，mysql/data目录下）
  - long_query_time 指定记录慢查询日志SQL执行时间得伐值（单位：秒，默认10秒）
  - log_queries_not_using_indexes  是否记录未使用索引的SQL
  - log_output 日志存放的地方【FILE,TABLE】，不要用table，用file即可

- 记录符合条件得SQL

  - 查询语句
  - 数据修改语句
  - 已经回滚得SQL  

- 解读

  ```sql
  # User@Host: root [root] @ localhost [127.0.0.1] Id: 10 --用户名 、用户的IP信息、线程ID号
  # Query_time: 0.001042 --执行花费的时间【单位：毫秒】
  #Lock_time: 0 .000000 --执行获得锁的时间
  #Rows_sent: 2 --获得的结果行数
  #Rows_examined: 2 --扫描的数据行数
  SET timestamp=1535462721; --这SQL执行的具体时间
  SELECT * FROM myarchive LIMIT 0，1000; --具体的SQL语句
  ```

### 分析

常用的慢查询日志分析工具

- mysqldumpslow：mysql/bin目录下的exe文件，登录到mysql服务端才能访问
  - 汇总除查询条件外其他完全相同的SQL，并将分析结果按照参数中所指定的顺序输出。
  - 语法
    - mysqldumpslow -s r -t 10  /slow-mysql.log
      -  -s order (c,t,l,r,at,al,ar) 
      - c:总次数
      - t:总时间
      - l:锁的时间
      - r:总数据行
      - at,al,ar  :t,l,r平均数  【例如：at = 总时间/总次数】
    -  -t  top   指定取前面几条作为结果输出
- pt_query_digest：详细内容见扩展阅读
  - per1 .\pt-query-digest  --explain h=127.0.0.1, u=root,p=password /slow-mysql.log。需要per1环境
  - 汇总的信息【总的查询时间】、【总的锁定时间】、【总的获取数据量】、【扫描的数据量】、【查询大小】
  - 结果解析
    - Response: 总的响应时间。
    - time: 该查询在本次分析中总的时间占比。
    - calls: 执行次数，即本次分析总共有多少条这种类型的查询语句。
    - R/Call: 平均每次执行的响应时间。
    - Item : 查询对象
  - 还有执行计划

## 索引与执行计划

### 索引

- 索引是什么
  - MySQL官方对索引的定义为：索引（Index）是帮助MySQL高效获取数据的数据结构。可以得到索引的本质：索引是数据结构。
- 索引得分类
  - 普通索引：即一个索引只包含单个列，一个表可以有多个单列索引
  - 唯一索引：索引列的值必须唯一，但允许有空值
  - 复合索引：即一个索引包含多个列
  - 聚集索引：就是索引与数据块顺序对应，比如id主键。索引键值的逻辑顺序与索引所服务的表中相应行的物理顺序相同的索引，被称为**聚集索引**，反之为**非聚集索引**，索引一般使用[二叉树](https://zh.wikipedia.org/wiki/二叉树)排序索引键值的，聚集索引的索引值是直接指向数据表对应元组的，而非聚集索引的索引值仍会指向下一个索引数据块，并不直接指向元组，因为还有一层索引进行重定向，所以非聚集索引可以拥有不同的键值排序而拥有多个不同的索引。而聚集索引因为与表的元组物理顺序一一对应，所以只有一种排序，即一个数据表只有一个聚集索引。
    - 聚簇索引(聚集索引)：并不是一种单独的索引类型，而是一种数据存储方式。具体细节取决于不同的实现，InnoDB的聚簇索引其实就是在同一个结构中保存了B-Tree索引(技术上来说是B+Tree)和数据行。
    - 聚集索引一般是表中的主键索引，如果表中没有显示指定主键，则会选择表中的第一个不允许为NULL的唯一索引，如果还是没有的话，就采用Innodb存储引擎为每行数据内置的6字节RowID作为聚集索引
  - 非聚簇索引：不是聚簇索引，就是非聚簇索引
    - show global variables like "%datadir%";
- 基础语法
  - 查看索引：SHOW INDEX FROM table_name\G
  - 创建修改索引：创建过多索引，会占用更多空间，并且虽然查询很快，但是增删改的效率都会收到影响，因为维护数据的同时还要维护索引。一般来说一个表索引不能超过5个
    - CREATE  [UNIQUE ] INDEX indexName ON mytable(columnname(length));
    - ALTER TABLE 表名 ADD  [UNIQUE ]  INDEX [indexName] ON (columnname(length)) 
  - 删除索引：DROP INDEX [indexName] ON mytable;
- 什么情况建索引：
  - 某一列相对来说较唯一
  - 经常用来查询显示的列
  - 经常用来关联的列，where条件中用到的列，以及join on用到的列
- 问题
  - MySQL中myisam与innodb的区别?
  - redo和undo干什么用的?
  - hash索弓|是什么，什么存储弓|擎支持?有什么优缺点?
  - btree和b + tree有什么样的区别，对于范围检索来说，b+ tree好在哪里?
  - 全文索引是怎么回事?
  - MySQL中InnoDB支持的四种事务隔离级别是什么?有什么区别
  - MYSQL中的间隙锁是怎么回事，有几种方式产生间隙锁?
  - 能谈谈mysq|实现读写分离的原理吗，和存储弓|擎有什么关系?

### 执行计划

- 什么是执行计划

  - 执行计划是什么
       使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈

- 语法
     Explain + SQL语句

- 执行计划的作用

  - 表的读取顺序
  - 数据读取操作的操作类型
  - 哪些索引可以使用
  - 哪些索引被实际使用
  - 表之间的引用
  - 每张表有多少行被优化器查询

- 执行计划详解

  - id：select查询的序列号,包含一组数字，表示查询中执行select子句或操作表的顺序

    - id相同，执行顺序由上至下
    - id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
    - id相同不同，同时存在

  - select_type：查询的类型，主要是用于区别 普通查询、联合查询、子查询 等的复杂查询

    - SIMPLE：简单的 select 查询，查询中不包含子查询或者UNION

      ```sql
      explain select * from t1
      ```

    - PRIMARY：查询中若包含任何复杂的子部分，**最外层查询**则被标记为PRIMARY

      ```sql
      explain select t1.*, (select t2.id from t2 where t2.id = 1 ) from t1
      ```

    - SUBQUERY：在SELECT或WHERE列表中包含了的**子查询**被标记为SUBQUERY

      ```sql
      explain select t1.*, (select t2.id from t2 where t2.id = 1 ) from t1
      ```

    - DERIUED：在FROM列表中包含的子查询被标记为DERIVED(衍生)，MySQL会递归执行这些子查询, 把结果放在临时表里。

      ```sql
      explain select t1.* from t1 , (select t2.* from t2 where t2.id =1 ) s2 where t1.id = s2.id --s2是衍生
      ```

    - UNION：若第二个SELECT出现在UNION之后，则被标记为UNION；
      若UNION包含在FROM子句的子查询中,外层SELECT将被标记为：DERIVED

      ```sql
      explain select * from t1 UNION select * from t2
      ```

    - UNI0N RESULT：从UNION表获取结果的SELECT

      ```sql
      explain select * from t1 UNION select * from t2
      ```

  - table：显示这一行的数据是关于哪张表的

  - type：type显示的是访问类型，是较为重要的一个指标，结果值从最好到最坏依次是

    - system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

    - 重点：system>const>eq_ref>ref>range>index>ALL

    - system：表只有一行记录（等于系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计

    - const：表示通过索引一次就找到了。const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL就能将该查询转换为一个常量

    - eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描

      ```sql
      EXPLAIN SELECT * from t1, t2 where t1.id=t2.id --通过id进行关联，id是索引且唯一，找id时出现eq_ref
      ```

    - ref：优化最多基本就到这里。**非唯一性索引**扫描，返回匹配某个**单独值**的所有行。本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体

    - range：最少优化到这里。只检索给定范围的行,使用一个索引来选择行。key 列显示使用了哪个索引
      一般就是在你的where语句中出现了between、<、>、in等的查询。这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束语另一点，不用扫描全部索引。

    - index：当查询的结果全部是索引列时，也是全表扫描，会扫面整个索引文件，但不扫描非索引数据

    - ALL：Full Table Scan，将遍历全表以找到匹配的行。即全表扫描

  - possible_keys：可能用到的索引

  - key：实际使用的索引。如果为NULL，则没有使用索引

    - 查询中若使用了覆盖索引，则该索引和查询的select字段重叠

    ```sql
    --除了id，有2个联合索引index_col1_col2、index_col1_col2_col3
    EXPLAIN select col1, col2 from t1 --possible_keys为null，key为index_col1_col2
    EXPLAIN select col1, col2 from t1 where col1=1 --possible_keys为index_col1_col2、index_col1_col2_col3，key为index_col1_col2。可见possible_keys会将包含有col1的所有索引列入
    ```

  - key_len

    - 表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好

    - key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的。如varchar(255)，即使只是用了1个字符，key_len也是255

    - key_len表示索引使用的字节数，根据这个值，就可以判断索引使用情况，特别是在组合索引的时候，判断所有的索引字段是否都被查询用到。

    - char和varchar跟字符编码也有密切的联系，latin1占用1个字节，gbk占用2个字节，utf8占用3个字节。（不同字符编码占用的存储空间不同）

    - 字符串类型：CHAR和VARCHAR类型类似,但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。这里我们只需要考虑CHAR、VARCHAR，因为不太可能用TEXT、BLOB来建立索引，大字段建索引想崩了？

      | 类型       | 大小                | 用途                          |
      | ---------- | ------------------- | ----------------------------- |
      | CHAR       | 0-255字节           | 定长字符串                    |
      | VARCHAR    | 0-65535字节         | 变长字符串                    |
      | TINYBLOB   | 0-255字节           | 不超过255个字符的二进制字符串 |
      | TINYTEXT   | 0-255字节           | 短文本字符串                  |
      | BLOB       | 0-65 535字节        | 二进制形式的长文本数据        |
      | TEXT       | 0-65 535字节        | 长文本数据                    |
      | MEDIUMBLOB | 0-16 777 215字节    | 二进制形式的中等长度文本数据  |
      | MEDIUMTEXT | 0-16 777 215字节    | 中等长度文本数据              |
      | LONGBLOB   | 0-4 294 967 295字节 | 二进制形式的极大文本数据      |
      | LONGTEXT   | 0-4 294 967 295字节 | 极大文本数据                  |

      ```sql
      explain select * from s2 where name= 'enjoy' ; --比如 utf8 下 name char(10)，其它编码计算方法一致，把3换成对应字节数即可
      ```

      - 索引字段为char类型+不可为Null时：key_len=3*10=30
      - 索引字段为char类型+允许为Null时：key_len=3*10+1=31，1个字节记录该字段可为null
      - 索引字段为varchar类型+不可为Null时：key_len=3*10+2=32，varchar是可变长度的，2个字节记录varchar实际使用了多少字节
      - 索引字段为varchar类型+允许为Null时：key_len=3*10+2+1=32，varchar是可变长度的，2个字节记录varchar实际使用了多少字节，1字节记录该字段可为null

    - 数值类型：key_len=类型字节数+可为null标记等

      | 类型         | 大小  | 范围(有符号)                                                 | 范围(无符号)                                                 | 用途           |
      | ------------ | ----- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------- |
      | TINYINT      | 1字节 | (-128, 127)                                                  | (0,255)                                                      | 小整数值       |
      | SMALLINT     | 2字节 | (-32 768，32 767)                                            | (0，65535)                                                   | 大整数值       |
      | MEDIUMINT    | 3字节 | (-8388608,8388607)                                           | (0, 16777 215)                                               | 大整数值       |
      | INT或INTEGER | 4字节 | (-2147483648，2147 483 647)                                  | (0，4294 967 295)                                            | 大整数值       |
      | BIGINT       | 8字节 | (-9 233 372036 854775 808，9223 372036 854 775 807)          | (0, 18446 744 073709 551 615)                                | 极大整数值     |
      | FLOAT        | 4字节 | (-3.402 823 466E+38 , -1.175 494 351E-38)，0, (1.175 494351 E-38，3.402 823466 351 E+38) | 0，(1.175 494 351 E-38，3.402 823 466E+38)                   | 单精度浮点数值 |
      | DOUBLE       | 8字节 | (-1.797 693 134 8623157 E+308 , -2.225073 858507201 4 E-308), 0, (2.225 073858 507201 4 E-308 ,1.797 693 134 862 3157 E+308) | 0, (2.225 073 858507 201 4 E-308 ,1.797 693 134 8623157 E+308) | 双精度浮点数值 |

    - 时间类型：

      | 类型      | 大小(字节)                        | 范围                                    | 格式<br/>          | 用途                    |
      | --------- | --------------------------------- | --------------------------------------- | ------------------ | ----------------------- |
      | DATE      | 3                                 | 1000-01-01~9999-12-31                   | YYYY-MM-DD         | 日期值                  |
      | TIME      | 3                                 | -838:59:59~838:59:59                    | HH:MM:SS           | 时间值或持续时间        |
      | YEAR      | 1                                 | 1901~2155                               | YYYY               | 年份值                  |
      | DATETIME  | 5.6版本及以后为5字节，之前为8字节 | 1000-01-01 00:00:00~9999-12-31 23:59:59 | YYYY-MM-DDHH:MM:SS | 混合日期和时间值        |
      | TIMESTAMP | 4                                 | 1970-01-01 00:00:00~2037年某时          | YYYYMMDDHHMMSS     | 混合日期和时间值,时间戳 |

    - 总结：NOT NULL=字段本身的字段长度，NULL=字段本身的字段长度+1(因为需要有是否为空的标记，这个标记需要占用1个字节)

      - 变长字段需要额外的2个字节（VARCHAR值保存时只保存需要的字符数，另加一个字节来记录长度(如果列声明的长度超过255，则使用两个字节)，所以VARCAHR索引长度计算时候要加2），固定长度字段不需要额外的字节。
      - 而NULL都需要1个字节的额外空间,所以索引字段最好不要为NULL，因为NULL让统计更加复杂并且需要额外的存储空间。
      - 复合索引有最左前缀的特性，如果复合索引能全部使用上，则是复合索引字段的索引长度之和，这也可以用来判定复合索引是否部分使用，还是全部使用。

  - ref：显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值

    - const：及表示常量，如where name="yuanya"
    - 索引名：格式为 schema_name.table_name.column_name，如yuanya.user.id

  - rous：根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数。同一个sql，有无索引，扫描的行数明显将不一样，有索引可以扫描行数更少，扫描行数越少即执行效率越高

  - Extra：包含不适合在其他列中显示但**十分重要**的额外信息

    - Using filesort：说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。
      MySQL中无法利用索引完成的排序操作称为“文件排序"

      ```sql
      EXPLAIN select cl from tl whe re c1='ac' order by c3 --索引是c1_c2和c1_c2_c3，没有用到索引排序，将Using filesort
      EXPLAIN select cl from tl whe re c1='ac' order by c2,c3 --优化，将用到c1_c2_c3索引排序
      ```

    - Using temporary：使了用临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序 order by 和分组查询 group by

      ```sql
      EXPLAIN SELECT coll FROM tl WHERE col1 IN ('ac', 'ab', 'aa' ) GROUP BY c2 --Using temporary
      EXPLAIN SELECT coll FROM tl WHERE col1 IN ('ac', 'ab', 'aa' ) GROUP BY c1, c2 --优化，最左前缀原则
      ```

    - USING index：是否用了覆盖索引。表示相应的select操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，效率不错！如果同时出现using where，表明索引被用来执行索引键值的查找;

      - 聚集索引（主键索引）：聚集索引就是按照每张表的主键构造一颗B+树，同时叶子节点中存放的即为整张表的记录数据。聚集索引的叶子节点称为数据页，聚集索引的这个特性决定了索引组织表中的数据也是索引的一部分。
      - 辅助索引（二级索引）：非主键索引，叶子节点=键值+书签。Innodb存储引擎的书签就是相应行数据的主键索引值。
      - 覆盖索引（索引覆盖）
        - 就是select的数据列只用从索引中就能够取得，不必从数据表中读取，换句话说查询列要被所使用的索引覆盖
        - 索引是高效找到行的一个方法，当能通过检索索引就可以读取想要的数据，那就不需要再到数据表中读取行了。如果一个索引包含了（或覆盖了）满足查询语句中字段与条件的数据就叫 做覆盖索引。
        - 是非聚集组合索引的一种形式，它包括在查询里的Select、Join和Where子句用到的所有列（即建立索引的字段正好是覆盖查询语句[select子句]与查询条件[Where子句]中所涉及的字段，也即，索引包含了查询正在查找的所有数据）。
        - 不是所有类型的索引都可以成为覆盖索引。覆盖索引必须要存储索引的列，而哈希索引、空间索引和全文索引等都不存储索引列的值，所以MySQL只能使用B-Tree索引做覆盖索引

    - Using where：表明使用了where过滤

    - Using join buffer：使用了连接缓存，即使用join on连接查询，连接查询会使用join buffer

    - Impossible where：where子句的值总是false，不能用来获取任何元组

## sql优化

假设联合索引 index_c1_c2_c3

1. 尽量全值匹配：即尽量使联合索引中更多的列被使用

   ```sql
   EXPLAIN SELECT * FROM staffs WHERE c1 = 'July'; --根据索引第一个列找到后就在该值下全扫描了
   EXPLAIN SELECT * FROM staffs WHERE c1 = 'July' AND c2 = 25; --第一个列找到后继续folow索引第二个列
   EXPLAIN SELECT * FROM staffs WHERE c1 = 'July' AND c2 = 25 AND c3 = 'dev'; --同理继续第三个列
   ```

2. 最佳左前缀法则：如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列。最左前缀原则其实就是防止索引失效的原则

   ```sql
   --没有问题，c1 c2 c3依序索引生效，and连接的条件是无序的，谁前谁后都不影响索引
   EXPLAIN SELECT * FROM user where c1="xxx" and c2="xxx" where c3="xxx"
   --c1被砍掉了，等同于失去head的链表无从循迹，失去火车头的火车跑不起来，索引失效
   EXPLAIN SELECT * FROM user where c2="xxx" where c3="xxx"
   --c1仍然可以索引生效，但是没有c2，c3也无从生效
   EXPLAIN SELECT * FROM user where c1="xxx" where c3="xxx"
   ```

3. 不在索引列上做任何操作（计算、函数、(自动or手动)类型转换），会导致索引失效而转向全表扫描

   ```sql
   --c1仍然生效，c2做了操作，将失效，c3自然也失效
   EXPLAIN SELECT * FROM user where c1="xxx" and left(c2,3)=23 where c3="xxx"
   ```

4. 范围条件放最后：存储引擎不能使用索引中范围条件右边的列。in查询是范围查询的一种

   ```sql
   --c1仍然生效，查询c2为23的过程c2是作为索引生效了的，因为b+tree的结构，在c1="xxx"上c2是有序的，索引到23之后，开始转为扫描c2>23所有数据，c2开始失效，c3则完全失效
   EXPLAIN SELECT * FROM user where c1="xxx" and c2>23 where c3="xxx"
   ```

5. 覆盖索引尽量用：尽量使用覆盖索引（只访问索引的查询，即索引列和查询列一致），减少select *。覆盖索引可以仅遍历索引就获得数据，过程中不需要取访问非索引数据内容

   ```sql
   EXPLAIN SELECT c1, c2, c3 FROM user where c2=23
   ```

6. 不等于要慎用：mysql 在使用不等于(!= 或者<>)的时候无法使用索引会导致全表扫描

   ```sql
   EXPLAIN SELECT c1, c2, c3 FROM c1, c2, c3 where c2!=23 --但是覆盖索引可以使使用!=的语句仍使用索引
   ```

7. Null/Not Null对索引可能有影响：

   ```sql
   EXPLAIN SELECT * FROM user where c1 is null --如果c1的定义是可以为null，那么索引生效，如果c1的定义是not null那么，索引不生效
   EXPLAIN SELECT * FROM user where c1 is not null --索引失效
   EXPLAIN SELECT c1, c2, c3 FROM use where c1 is not null --覆盖索引可以使使用null/Not Null的语句仍索引生效
   ```

8. Like查询要当心：like以通配符开头('%abc...')mysql索引失效会变成全表扫描的操作

   ```sql
   EXPLAIN SELECT * FROM user where c1 like "%asd%"; --失效
   EXPLAIN SELECT c1, c2 FROM user where c1 like "%asd%"; --覆盖索引可使索引生效
   EXPLAIN SELECT * FROM user where c1 like "asd%"; --生效，b+tree是有序的，从前往后的
   ```

9. 字符类型加引号：字符串不加单引号索引失效

   ```sql
   EXPLAIN SELECT * FROM user where c1=917; --失效。覆盖索引。。。算了不说了
   EXPLAIN SELECT * FROM user where c1='917'; --生效
   ```

10. OR改UNION效率高

    ```sql
    EXPLAIN SELECT * FROM user where c1='x' or c1="xxx"; --失效。覆盖索引……
    EXPLAIN SELECT * FROM user where c1='x' UNION SELECT * FROM user where c1="xxx"; --生效。大概in查询也可以这么玩？如果in中数据很多？
    ```



##### insert语句优化

- 提交前关闭自动提交

- 尽量使用批量insert语句

- 可以使用MyISAM存储引擎

- 例：有一个sql文件，里面几百万insert语句，用java  dbc写入，读取sql文件可以

  - 读一行插入一行，效率极低

  - 使用batch，关闭自动提交，每几千行执行一次这样，效率稍高，但是batch实际上还是一条一条执行的

  - 将数据库改为MyISAM存储引擎，将sql拼接成如下形式，可以极大节省效率，同样几千行执行一次即可，执行完后再将数据库改回innodb，注意仅导入数据时这么做，生产环境一定还是不能MyISAM

    ```sql
    insert into user(c1,c2) value(1,2)(2,3)(3,4)
    ```

  - LOAD DATA INFLIE：使用LOAD DATA INFLIE ,比一般的insert语句快几十倍不止，几百万数据也就几秒完成

    ```sql
    select * into OUTFILE 'D:\\product.txt' from t1; --导出t1到D:\\product.txt
    create table t2 select * from t1 where 1=2; --仅拷贝表结构，如果1=1或者不加where则包含数据
    load data INFILE 'D:\\product.txt' into table t2; --导入D:\\product.txt到t2
    ```



最后一题

100w数据，索引是date_str，25秒左右

```sql
SELECT a.date_str, a.shopCode, a.add_car_pv
FROM dp_car_copy a
WHERE  1=1
AND (
    (
        a.date_str >= '2018-07-01'
        AND a.date_str <= '2018-08-30'
    )
)
ORDER BY a.date_str, a.shopCode, a.add_car_pv desc
```

1. 索引覆盖，将3个列建为联合索引，15秒左右
2. 查看执行计划，发现type已经是range了，但是extra中using filesort，所以删除desc，使用索引本身的顺序，1秒左右
3. 如果确实需要desc，现在可能会想到索引反着建，但是如果下次又要正着查询呢，所以还是选择再动索引，而是在后台代码中处理即可，因为返回的是一个list，所以倒着取list不就好了。这里主要要学会变通，sql当然需要满足需求，但是不要忽视后台代码或者其它因素



limit 10000 10：他不会只扫描10000-10010，而是扫描前10010条数据，所以当数据很多时，取后面的数据效率很低，这时候就要通过其他方式解决，如假如id是自增无断缺的，那么可以where id>9999 and id<10011 limit 0,10



## mysql



###### 事务

```sql
set autocommit=0; --关闭自动提交
commit; --提交事务
```



任何数据库层面的优化都抵不上应用系统的优化

- Client：基于不同的语言不同的脚本都可以实现，甚至可以写socket直接去连接Server

- Server：即我们通常所说的MySQL。架构见图01、02

  - SQL_Layer

    - 初始化模块：Server启动时，加载默认参数，进行初始化。
    - 连接管理模块：启动完成后server将监听请求
    - 连接进程模块：请求进来之后分发到连接进程模块（connection pool，有任务进来了，有没有线程活跃，有则去服务这个连接，没有就创建一个新的连接。连接池不止是数据库外的Client有，Server内也有），完成数据库连接
    - 用户模块：鉴权，校验令牌，是否有权限
    - 命令分发器：辨别请求（请求有两种，Query、commond），如果发现查询缓存模块种有，那么就直接返回结果了，不会前往到下面的模块。没有缓存则继续往下交给命令解析器解析后分发
    - 查询缓存模块：query cache，它类似于一个hashmap，把Sql语句和参数做一个hash作为key，value就是结果值，这就是首次查询比重复查询慢的多（查询都有时间报告，可以注意到）。所以调大cache也是优化的一种，只对查询有效
    - 日志记录模块：
    - 命令解析器（parser）：解析完之后，分发到不同的模块处理
    - 以下是被分发的模块
      - 查询优化器：被分发select这类请求。这里的select包括crud
      - 表变更模块：被分发dml这类请求
      - 表维护模块：ddl
      - 复制模块：rep，主从复制等
      - 状态模块：status
    - 访问控制模块：上面所有被分发模块最终都到访问控制模块
    - 表管理模块：
    - 存储引擎接口：关键。与Storage Engines打交道
    - 还有独立的两个模块：
      - 核心API：内存管理、小IO、数字及字符串等，类似于jvm或是一个manager的一个功能
      - 网络交互：网络监听、交互协议处理等

  - Storage Engines。很多，主要关注innodb和myisam这两个存储引擎

    - 文件存储形式：每一个schama（mysql种即database）都有一个文件夹，不同的引擎创建的文件格式和数量都不一样，innodb创建的一个表是两个文件，user.frm、user.ibd(inno database dater)两个文件格式，myisam是frm、MYD、MYI格式共3个存储文件

      |          | inno db                                                      | MySIAM                                                       |
      | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
      | 存储文件 | .frm表定义文件<br />.ibd数据文件（索引和数据文件集在一起。聚集索引，基于主键展开的b+tree） | SYWWA<br/>.frm表定义文件<br/>.myd数据文件<br/>.myi索引文件（也是一个索引tree，每一个索引都有一个指向，就像一个文件系统一样，指向来快速定位数据） |
      | 锁       | 表锁、行锁                                                   | 表锁                                                         |
      | 事务     | ACID                                                         | 不支持                                                       |
      | CRDU     | 读、写                                                       | 读多                                                         |
      | count    | 扫表                                                         | 专门存储的地方，每个表中有多少行是被专门存储下来的，         |
      | 索引结构 | B+TREE                                                       | B+TREE                                                       |

  - 性能

    - 影响性能的因素
      - 人为因素：需求，是否是不是一定要这么去做
        - count(\*)：论坛分页需要用到，数据小于100w这样的数据，count(\*)还ok，但是如果是1000w上e，可能count(\*)就会成为一个瓶颈。这时候就要考虑数据是：实时、准实时、可以有误差。我们知道MySIAM每个表中有多少行是被专门存储，count(\*)只需要直接取出来就ok，那么我们也可以学习它专门找一个地方来存储，如果数据只需要准实时、或者可以有误差，就可以这么做
      - 程序员因素：比如太过面向对象
        - 有一个相册、相册里有一个相片，相片有评论数。要看到相册里前三个照片的评论总数。可能会这么做：方案1，在相片中limit出来前n，然后再遍历这n个pohoto_id去评论中count出来，这很面向对象，但进行了1+n次数据库io；方案2，limit出来前n，再通过pohoto_id的集合去评论中做一次 count(*) group by pohoto_id集合，无论n是多少都是2次io，要好一些；
      - cache：
        - server内部的查询cache：最多也就放大一点，不怎么管得到。
        - 外部有redis、menmerycache等，如果设计的不够好，所有的请求都打到数据库层面，也会有性能问题
      - 可扩展：过度追求，如过度设计冗余字段等
        - 先看一个合理的扩展设计：如一个user_id给另一个user_id发了一个msg，那么就是两个id和一个msg内容 就是表结构，因为我们在msg外部界面 消息列表 只需要知道谁给谁发了消息，列表可能很多，msg内容可能是一个大字段，只是看一个简略列表就要连带到这么多大字段内容，很影响性能，那么为了点进去才需要知道msg的具体内容，msg内容就可以再拆出来一个表，之前的表就变成两个userid和一个msgid，需要看消息内容再根据id去查
        - 过度：如订单表，首先一般是有userid的，要不要冗余一个用户姓名，可能会展示订单的时候更方便，这就属于没有必要，因为查看订单详情的时候连带查询一个用户信息并不是很耗费性能的事情
      - 表范式
      - 应用场景：
        - OLTP：On-Line Transaction Processioning 链接事务处理。一般的普通系统基本都是
          - 特点
            - 数据量大
            - 每次访问数据比较少
            - 数据离散
            - 活跃数据占比不大
          - 优化
            - 活跃数据占比不大：扩大内存容量将活跃数据cache住
            - IO频繁 IOPS（per second）：频繁，一般单次数据量就不会很大，Not 吞吐量，不需要关注吞吐量
            - 并发大：CPU要强劲
            - 与客户端交互频繁：网络设备扛流量的能力不能差
        - OLAP：On-Line Analysis Processioning  链接分解处理。一般是数据仓库，数据分析的系统
          - 优化
            - 数据量大
            - 并发不高
            - 单次检索数据量多
            - 数据访问集中
            - 没有明显的活跃数据
          - 优化
            - 磁盘单位容量要大
            - 单次检索数据量多：关注IO吞吐量，Not IOPS
            - 并发少：CPU要求没那么高
            - 计算量大时间长、并行要求高：做集群，集群同步会网络通讯要求高。这就是为什么大数据都是集群...
    - 提高性能：
      - 索引：一种数据结构，高效定位数据。select * from table where id =1，如果没有索引，就会去数据文件轮询每一条数据，对比得到，没有索引第一个读到的不一定是1，因为物理磁盘上的数据存储不是逻辑上那么连续的，所以就需要索引。如文件系统(柱面，磁道，扇区)：我们去找文件的时候，我们的机械硬盘磁头去找到文件，是我们文件系统有记录，通过柱面、磁道、扇区定位到文件在磁盘哪个地方哪个扇区，磁头就可以直接去到文件位置读取。索引table.ibd也是类似
      - 索引。衡量索引，主要考虑IO渐进复杂度，即IO越来越多，索引是否能还是那么高效。命名好习惯：idx\_作为索引命名前缀，还有如唯一约束以uni\_开头等
        - 种类
          - Hash：有冲突，只能做等值查询无法做范围查询如where id>1
          - Full Text：做全文搜索
          - 前缀索引：如一个string，截取其前面几个字符做索引
          - R-Tree：空间索引，如3公里以内的商店。替代方案GEOHash
          - B+Tree：红黑树高度不可控，那么查询就非常耗时
            - 联合索引：（id,age）作为索引，且遵循最左前缀匹配的规则，先从左边id开始寻找索引，如果where age=10 and id=1，这样是找不到索引的。联合索引还有一个值得注意的点，B+Tree下，因为id相同的索引是聚集一起的一定区域，在id相同的情况下，第二列age也是是排序的，这和mysql中B+Tree插入联合索引的设计有关，那么这个时候我们通过where id=1 order by age，得到的结果是不用经过排序直接取出来的，因为本身就是按序的。这就是索引对分组排序的效率音响的点
        - pros
          - 提高检索效率
          - 降低排序成本：排序分组主要小号的是内存和cpu资源
        - cons
          - 更新索引的IO量
          - 调整索引所致的计算量
          - 存储空间
        - 是否创建索引
          - 较频繁的作为查询条件的字段应该创建索引
          - 唯一性太差的字段不适合单独创建索引
          - 更新非常频繁的字段不适合创建索引
          - 不会出现在where子句中的字段不该创建索引

## 锁

- row-level：
  - pros 
    - 粒度小
  - cons
    - 获取、释放所做的工作更多
    - 容易发生死锁：如果事务A、B的第一个操作同时执行，导致t1和t2的id=1同时被行锁
      - 事务A：update table1 where id=1，t1的id=1行锁了，update table2 where id=1，要等待t2的id=1解锁
      - 事务B：update table2 where id=1，t2的id=1行锁了，update table1 where id=1，要等待t1的id=1解锁
      - 相互等待构成死锁
  - 实现Innodb的锁，以下锁都是系统底层将帮我们做，以下只是现实演示，实际中不会用这些sql的
    - 共享锁：
      - 读锁：lock table user read;上读锁，读可以无阻塞共享，只有写需要等待阻塞，unlock tables释放所有锁
      - 写锁：lock table user write;，只有写可以无阻塞同时进行，
    - 排他锁
    - 间隙锁：通过在指向数据记录的第一个索引键之前和最后一个索引键之后的空域空间上标记锁定信息来实现的。是一个范围的行锁，如update user set nick name = "Xxx" where id >1 and id<4;将锁住 1-4之间的行(假如id是递增的)
    - 锁优化
      - 尽可能让所有的数据检索都通过索引来完成
      - 合理设计索引
      - 减少基于范围的数据是检索过滤条件
      - 尽量控制事务的大小
      - 业务允许的情况下，尽量使用较低级别的事务隔离
- table-level：
  - pros
    - 实现逻辑简单
    - 获取、释放快
    - 避免死锁：因为直接获取不到整张表。。。没讲清楚，好像是两个表都锁了的情况下，先去另一个表的请求获取不到表直接out不会wait，它之前锁的表就解锁了，另一个事务的后一个请求就可以继续操作了
  - cons：粒度太大，并发不够高
  - 实现MyISAM
- 页锁：一个表很大是会分为多个page，所以page是table的一部分，包含大于1个row。innodb不支持page锁

|                | 共享锁(S) | 排他锁(X) | 意向共享锁(IS) | 意向排他锁(IX) |
| -------------- | --------- | --------- | -------------- | -------------- |
| 共享锁(S)      | 兼容      | 冲突      | 兼容           | 冲突           |
| 排他锁(X)      | 冲突      | 冲突      | 冲突           | 冲突           |
| 意向共享锁(IS) | 兼容      | 冲突      | 兼容           | 兼容           |
| 意向排他锁(IX) | 冲突      | 冲突      | 兼容           | 兼容           |

## 优化

- 原则

  - 永远用小结果集驱动大结果集
    - join表
  - 只取出自己需要的columns
    - 数据量
    - 排序占用空间：max length for sort data
  - 仅仅使用最有效的过滤条件：key length
  - 尽可能避免复杂的join和子查询：会锁资源

- QEP：query excution plan，查询执行计划，由查询优化器选择得到的查询最佳路径

  - 使用：在select语句前面加个explain就可以查看该sql的执行计划
    - 如explain SELECT * FROM user order by user.id
    - 如果是update user where id=5，只需要换成select语句即可，explain select * from user where id=5，因为修改也是要先找到才修改嗷
    - 字段：
      -  ID: Query Opt imizer所选定的执行计划中查询的序列号，一个计划的唯一标识符，一个计划可能有多行组成（比如join的查询），所以id是可重复的
      -  select_type：查询类型，类型很多具体百度对照
         - DEPENDENT SUBQUERY: 子查询中内层的第一个SELECT,
      -  table：当前行执行的表是哪个表，同一个计划多行中涉及的表可能也不同（比如join的查询），一般会有一个驱动表，驱动表即第一个要操作查询的表，这是查询优化器帮我们选出来的，也是满足小结果集驱动大结果集，如计划中涉及一个table有100个数据， 另一个table有1w的数据，那么第一个table是被选作驱动表的，在计划中也是第一行，除了第一行计划的多行并不是按顺序的
      -  type：重中之重。对表所使用的访问方式，类型很多，具体介绍百度一下或者官网看看。依次从好到差: System、const、eq_ref、ref、fulltext、ref or null、unique_subquery , index_subquery , range、index_merge、index、ALL
      -  possible_ keys：重点。可能命中的索引有哪些
      -  key：最终用到的是哪些索引，多个则是组合索引，满足最左前缀的点
      -  key_len：用到的索引长度，将选key_len最短的
      -  ref：过滤形式
      -  rows：该计划行查询数据所读取的行数，这就是为什么要小表驱动大表，小表100行，那么计划第一行最多读取100行就能完成计划第一行筛选，如果大表1w行作为驱动，那么计划第一行最多可能要1w读取1w行才能完成第一部分筛选
      -  filtered：
      -  Extra：用到了的资源，如where关键词、索引、排序、临时表之类的，如两表关联order by，两表需要关联并拷贝到一个临时表中进行排序

- profiling

  - 先set profiling=1;开启，
  - 然后执行任意查询语句select * from user order by age，
  - 然后再执行show profiles，可以看到一个表，是我们最近一次查询语句执行查询的过程操作查询，这些操作数据行都有id，
  - 可以通过id来执行最后的show profile cpu,block io for query 75，75即我们要查操作行的id，可以看到该操作的每一个过程状态所持续的时间，如初始化、排序之类的

- join：如A.id=B.id，A表是驱动表（是底层查询优化器自己选出来的），那么则遍历A筛选出来的这些A.id，到B中又遍历去找到对应的B.id值，即一个双层for循环，然后拼凑出结果到临时表。小结果集驱动大结果集，如果A是小表只筛选出来100条，那么只需要轮询B100次，如果A是大表筛选出来1w条，那么要轮询B1w次，虽然m\*n和n\*m是一样的，但是对B表的读取扫描次数是不一样的。show variables like 'ioin_ %';查看

  - Nested Loop Join
  - 优化
    - 永远用小结果集驱动大的结果集，这是查询优化器筛选的，我们只能改变强制使用某些索引，来一定程度影响筛选，筛选还是不可控的
    - 保证被驱动表上的Join条件字段已经被索引：这样每次扫描都会更快，也会放大小结果集驱动大结果集的优化效果
    - Join Buffer：join_buffer_size，当join表时，一些中间结果是缓存在Join Buffer中的，如三表join，A和Bjoin得到一个结果缓存到Join Buffer形成一个中间结果表，中间表再与C表join得到结果集，Join Buffer的使用是由底层决定的，并不是多表join一定会用到，调大join_buffer_size是一种预优化，给可能用到容量缓存的时候准备。因为如果Join Buffer容量不够就会缓存到磁盘分段，产生io

- order by

  - 实现
    - 有序，命中索引：因为B+tree索引是有序的结构，所以根据索引order by，执行计划的Extra中将不会出现用到排序的信息，因为有序直接拿出来就完了，不进行排序
    - 无序，未命中索引：show variables like '%sort%" ;可以找到sort_buffer_size。下面两种方式是底层选择的，当sort_buffer_size足够承载要返回的数据，那么则选用第二种，否则第一种。这也是为什么取数据尽量只取需要的Colum，而不要用select *，当然这只是一个点
      - 一：只把要排序的字段和指针copy到sort buffer中去排序，这一列指针指向数据表所在行，做完排序再根据指针去拿到数据表中的数据copy到结果集。共两次磁盘io
      - 二：直接copy所有要返回的字段和指针到sort buffer，之后又做一次拆分，将要排序的字段和指针copy出来，这时候指针就是指向copy出来的那些行了，之后与第一种一样。这样除了第一次copy的一次io，之后都在内存中操作了。节省io，但是耗费内存 ，空间换时间
  - 优化
    - 索引顺序致的话不需要再排序：可以看到索引是最主要的，可以直接索引没有排序后面的优化都不需要了
    - 加大max length for sort data从而使用第二种排序方法（排序只针对需要排序的字段）
    - 内存不充足时去掉不必要的返回字段
    - 增大sort buffer size：减少在排序过程中对需要排序的数据进行分段

- group by：基于order by，先排序后分组，所以适用于order by的优化也适用于group by

- distinct：基于group by，先排序后分组再去重，所以。。。不说了

- limit：当offset很大时，很慢。因为limit执行取出的数据不是size条，而是offset+size条

  - 自增无空缺的id可以SELECT * FROM user where id>10000 limnit 10;，总之就是根据业务通过其它方式筛选后再limit

- slow sql

  - 配置

    ```sql
    [mysqld]
    slow_query_log=1
    slow_query_log_file=/path/to/file
    long_query_time=0.2
    log_output=FILE
    ```

  - show full processlist;，然后slow sql来定位sql，最后explain查看分析



- 建索引的几大原则
  - 最左前缀匹配原则，非常重要的原则, mysq|会直向右匹配直到遇到范围查询(>、<、 between、 like)就停止匹配，比如a= 1 andb= 2andc> 3 andd = 4如果建立(a,b,c,d)顺序的索引, d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。
  - =和in可以乱序，比如a= 1 andb= 2andc= 3建立(a,b,c)索引可以任意顺序, mysql的查询优化器会帮你优化成索引可以识别的形式
  - 尽量选择区分度高的列作为索引,区分度的公式是count(distinct co)/count(*)，表示字段不重复的比例,比例越大我们扫描的记录数越少，唯一键的区分度是1 ，而一些状态、性别字段可能在大数据面前区分度就是0 ,那可能有人会问，这个比例有什么经验值吗?使用场景不同，这个值也很难确定,一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录
  - 索引列不能参与计算,保持列"干净”, 比如from_ unixtime(create_ time) =’2014-05-29' 就不能使用到索引,原因很简单，b+树中存的都是数据表中的字段值，但进行检索时, 需要把所有元素都应用函数才能比较,显然成本太大。所以语句应该写成create_ time = unix_ timestamp(' 2014-05-29' );
  - 尽量的扩展索引,不要新建索引。比如表中已经有a的索引,现在要加(a,b)的索引，那么只需要修改原来的索引即可

