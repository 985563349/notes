## MySQL基础命令

操作系统：Mac OS

MySQL版本：8.0.28



### 服务器程序

#### 启动服务器

```shell
mysql.server start
```



#### 关闭服务器

```shell
mysql.server stop
```



#### 连接服务器

```shell
mysql -h主机名  -u用户名 -p密码
```

各参数意义如下：

| 参数名 | 含义                                                         |
| :----: | ------------------------------------------------------------ |
|   -h   | 启动服务器程序的计算机的域名或IP地址，如果服务器程序运行在本机，可以省略该参数，也可填入`localhos`或`127.0.0.1`。也可写作`--host=主机`的形式。 |
|   -u   | 用户名，超级管理员用户名是`root`。也可写作`--user=用户名`的形式。 |
|   -p   | 密码，超级管理员无初始密码。也写作`--password=密码`的形式。  |



#### 断开服务器连接

如果想断开服务器连接，可在`mysql>` 提示符后输入以下任意指令：

```mysql
quit

exit

\q
```



------



### MySQL语句

在书写MySQL命令时需要注意以下几点：

- 大小写

  MySQL默认对命令的大小写没有限制。

- 字符串表示

  命令中的字符串可以使用单引号，或者双引号包裹

- 命令结束符号

  命令需要以以下几种符号之一结尾：

  - `;`
  - `\g`
  - `\G`

- 命令可以随意换行

  按回车键时输入的命令没有结束符号，命令不会提交。

- 可以一次提交多个命令

  多个命令之间使用结束符分割。

- 使用`\c`放弃本次操作

  如果想放弃本次编写的命令，可以输入`\c`结尾。



#### 数据库

##### 展示数据库

```mysql
SHOW DATABASES;
```



##### 创建数据库

```mysql
CREATE DATABASE 数据库名;
```



##### IF NOT EXISTS

在数据库已经存在的情况下使用`CREATE DATABASE`去创建这个数据库会产生错误。如果不清楚数据库是否存在，可以使用以下语句创建。

```mysql
CREATE DATABASE IF NOT EXISTS 数据库名;
```

该命令执行如果数据库不存在就创建它，否则不执行任何操作。



##### 查看当前数据库

```mysql
SELECT DATABASE();
```



##### 切换当前数据库

```mysql
USE 数据库名;
```



##### 删除数据库

```mysql
DROP DATABASE 数据库名;
```



##### IF EXISTS

若某个数据库不存在，调用`DROP DATABASE`命令去删除会产生错误。

```mysql
DROP DATABASE IF EXISTS 数据库名;
```

该命令执行如果数据库存在就删除它，否则不执行任何操作。



#### 表

##### 展示表

```mysql
SHOW TABLES;
```



##### 创建表

```mysql
CREATE TABLE 表名 (
  列名1  数据类型  [列的属性],
  列名2  数据类型  [列的属性],
  ...
  列名n  数据类型  [列的属性],
) COMMENT '表注释';
```



##### IF NOT EXISTS

在表已经存在的情况下使用`CREATE TABLE`去创建这个表会产生错误。如果不清楚表是否存在，可以使用以下语句创建。

```mysql
CREATE TABLE IF NOT EXISTS 表名 (
  各列信息....
);
```

该命令执行若表不存在就创建它，否则不执行任何操作。



##### 删除表

```mysql
DROP TABLE 表1, 表2, ..., 表n;
```



##### IF EXISTS

若某个表不存在，调用`DROP TABLE`命令去删除会产生错误。

```mysql
DROP TABLE IF EXISTS 表名;
```

该命令执行如果表存在就删除它，否则不执行任何操作。



##### 查看表结构

以下命令会以表格的形式输出表结构，且输出内容相同：

```mysql
SHOW CREATE TABLE 表名;

DESC 表名;

EXPLAIN 表名;

SHOW COLUMNS FROM 表名;

SHOW FIELDS FROM 表名;
```

以下命令会以表创建命令的形式输出表结构：

```mysql
SHOW CREATE TABLE 表名;

SHOW CREATE TABLE 表名\G
```



##### 未选择数据库时对表操作

如果没有使用`USE`语句选择当前数据库，或在一条语句中遇到表分散在不同数据库，可以在语句中显示指定数据库。

```mysql
# 展示指定数据库中的表
SHOW TABLES FROM 数据库名;

# 指定表所属数据库
SHOW CREATE TABLE 数据库名.表名;
```



##### 修改表

###### 修改表名称

```mysql
ALTER TABLE 旧表名 RENAME TO 新表名;
```



###### 批量修改表名

```mysql
RENAME TABLE 旧表名1 TO 新表名, 旧表名2 TO 新表名2, ..., 旧表名2 TO 新表名n;
```



###### 转移表

在修改表名时指定数据库名，可以将表转移到对应数据库下。

```mysql
ALTER TABLE 表名 RENAME TO 数据库名.表名;
```



###### 新增列

```mysql
# 新增一列至列尾
ALTER TABLE 表名 ADD COLUMN 列名 数据类型 [列属性];

# 新增一列至列头
ALTER TABLE 表名 ADD COLUMN 列名 数据类型 [列属性] FIRST;

# 新增一列到指定列尾
ALTER TABLE 表名 ADD COLUMN 列名 数据类型 [列熟悉] AFTER 指定列名;
```



###### 删除列

```mysql
ALTER TABLE 表名 DROP COLUMN 列名;
```



###### 修改列

修改列时需要注意，修改后的数据类型和属性必须要兼容表中现有的数据。

```mysql
# 修改列数据类型和属性
ALTER TABLE 表名 MODIFY 列名 新数据类型 [新属性];

# 修改列名、数据类型和属性
ALTER TABLE 表名 CHANGE 旧列名 新列名 新数据类型 [新属性];

# 将列设为表第一列
ALTER TABLE 表名 MODIFY 列名 数据类型 属性 FIRST;

# 将列放置到指定列后
ALTER TABLE 表名 MODIFY 列名 数据类型 属性 AFTER 指定列名;
```



###### 一条语句多个操作

如果对同一个表有多个修改操作，可以把它们放到同一条语句中执行。

```mysql
ALTER TABLE 表名 操作1, 操作2, ..., 操作n;
```

