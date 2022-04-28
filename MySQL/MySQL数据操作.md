### MySQL数据操作

#### 插入

##### 插入单条数据

```mysql
INSERT INTO 表名(列1, ..., 列n) VALUES(列1的值, ..., 列n的值);
```



##### 批量插入数据

```mysql
INSERT INTO 表名(列1, ..., 列n) VALUES(列1的值, ..., 列n的值), (列1的值, ..., 列n的值), ...;  
```



#### 查询

##### 查询单列

```mysql
SELECT 列名 FROM 表名;
```



##### 查询多列

如果想查询多列数据，可以在`SELECT`后输入多个列名，列名间用逗号`,`隔开即可。

```mysql
SELECT 列名1, 列名2, ..., 列名n FROM 表名;
```



##### 列别名

查询结果集的列可以重新定义一个`别名`，命令如下：

```mysql
# 单列指定别名
SELECT 列名 [AS] 列别名 FROM 表名;

# 多列指定别名
SELECT 列名1 [AS] 列别名1, ..., 列名n [AS] 列别名n FROM 表名;
```

别名只影响本次展示结果集列名，并不会改变真实表中的列名。



##### 查询所有列

查询所有列通常会降低性能，如果不是必须需要所有列数据，否则不应该使用星号`*`查询所有列。

```mysql
SELECT * FROM 表名;
```



##### 查询结果去重

```mysql
# 单列结果去重
SELECT DISTINCT 列名 FROM 表名;

# 多列结果去重
SELECT DISTINCT 列名1, ..., 列名n FROM 表名;
```



##### 限制查询结果条数

```mysql
SELECT 列名 FROM 表名 LIMIT 开始行, 限制条数;
```



##### 查询结果排序

MySQL默认会按数据底层存储的顺序返回数据，在数据经过更新或删除后，不明确指定按照什么顺序排序返回结果，可以认为数据的返回顺序是不确定的。

```mysql
# 按单列值排序
SELECT 列名 FROM 表名 ORDER BY 列名 ASC|DESC

# 按多列值排序
SELECT 列名1, 列名2 FROM 表名 ORDER BY 列1 ASC|DESC, 列2 ASC|DESC;
```

`ASC`指按`升序`排序，`DESC`指按`降序`排序，不指定排序方向，默认使用`ASC`。



##### 单条件查询

MySQL`搜索条件`需要放在`WHERE`子句中。

```mysql
SELECT 列名 FROM 表名 WHERE 搜索条件;
```

如搜索`student_info`表中`name`为`Jee`的学生。

```mysql
SELECT * FROM student_info WHERE name = 'Jee';
```

例中的`name = 'Jee'`就是搜索条件，搜索条件中的`=`称为`比较操作符`。除了`=`外，MySQL还提供了其他比较操作符，比如：

|   操作符    |          示例          |            描述             |
| :---------: | :--------------------: | :-------------------------: |
|      =      |         a = b          |           a等于b            |
|  <> 或 !=   |         a <> b         |          a不等于b           |
|      <      |         a < b          |           a小于b            |
|     <=      |         a <= b         |        a小于或等于b         |
|      >      |         a > b          |           a大于b            |
|     >=      |         a >= b         |        a大于或等于b         |
|   BETWEEN   |   a BETWEEN b AND c    |       满足b <= a <= c       |
| NOT BETWEEN | a NOT BETWEEN b AND c  |           不满足            |
|     IN      |   a IN (b1, b2, ...)   |  a是b1, b2, ... 中的某一个  |
|   NOT IN    | a NOT IN (b1, b2, ...) | a不是b1, b2, ... 中的某一个 |
|   IS NULL   |       a IS NULL        |         a的值是NULL         |
| IS NOT NULL |     a IS NOT NULL      |        a的值不是NULL        |

> 比较判断时不能直接用普通的操作符和NULL值比较，必须使用`IS NULL`或者`IS NOT NULL`！



##### 多条件查询

###### AND 操作符

使用`AND`操作符连接多个`搜索条件`，当记录满足所有条件时才将其加入结果集。

```mysql
SELECT 列名 FROM 表名 WHERE 搜索条件1 AND 搜索条件2;
```



###### OR操作符

使用`OR`操作符连接多个`搜索条件`，当记录满足其中一个条件时便可加入结果集。

```mysql
SELECT 列名 FROM 表名 WHERE 搜索条件1 OR 搜索条件2;
```



###### 复杂搜索条件组合

在`AND`操作符和`OR`操作符组合使用时，`AND`操作符的优先级会高于`OR`操作符。为了条件避免冲突可以使用`()`来显示指定搜索条件的检测顺序。

```mysql
SELECT * FROM 表名 WHERE (搜索条件1 OR 搜索条件2) AND 搜索条件3;
```



##### 模糊查询

MySQL中提供两个操作符来支持`模糊查询`:

|  操作符  |     示例     |   描述   |
| :------: | :----------: | :------: |
|   LIKE   |   a LIKE b   |  a匹配b  |
| NOT LIKE | a NOT LIKE b | a不匹配b |

当不能完整描述要查询的信息时，需要使用某个符号来替代这些模糊的信息，这个符号称为`通配符`。MySQL中支持一下两个通配符：



- `%`：代表任意一个字符串：

  比如需要查询`student_info`表中`name`以`J`开头的数据

  ```mysql
  SELECT * FROM student_info WHERE name LIKE 'J%';
  ```

  或者查询`student_info`表中`name`包含`e`字符的数据

  ```mysql
  SELECT * FROM student_info WHERE name LIKE '%e%';
  ```

- `-`：代表任意一个字符：

  有时知道要查询的字符串中有多少个字符，而使用`%`时匹配范围太大，就可以使用`_`来做通配符。

  ```mysql
  SELECT * FROM student_info WHERE name LIKE 'J_';
  
  SELECT * FROM student_info WHERE name LIKE '_m_';
  ```



当待匹配的字符串中本身就包含普通字符`%`或`_`时，可以使用转义符`\`进行转义。

```mysql
SELECT * FROM student_info WHERE name LIKE 'J\_';
```

