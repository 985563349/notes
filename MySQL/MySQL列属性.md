### MySQL列属性

#### 默认值

定义列时可给列设置一个`DEFAULT`属性，用于指定在插入数据时未设置列值时的默认值，如果不设置则默认为`null`。

```mysql
列名 数据类型 DEFAULT 默认值
```

例如将`first_table`的`second_column`列的默认值指定为`abc`：

```mysql
CREATE TABLE first_table (
  first_column INT,
  second_column VARCHAR(100) DEFAULT 'abc'
);
```



#### NOT NULL属性

若要求表中的某些列必须有值，不能存放`null`，可以使用`NOT NULL`来定义该列。

```mysql
列名 数据类型 NOT NULL
```

例如给`first_table`的`first_column`列定义一个`NOT NULL`:

```mysql
CREATE TABLE first_table (
  first_column INT NOT NULL,
  second_column VARCHAR(100) DEFAULT 'abc'
);
```



#### 主键

当一个表内可以通过某列或者某些列确定一条唯一的记录，那么就可以把这列或这些列称为`候选建`。一个表中可能有多个`候选键`，可以选择其中一个作为`主键`。一个表最多只能有一个`主键`且值不能重复，所以通过`主键`可以找到一条唯一的记录。

1、如果主键的只是单列，可以直接在该列后声明`PRIMARY KEY`。

```mysql
CREATE TABLE student_info (
  number INT PRIMARY KEY,
  name VARCHAR(5),
  gender ENUM('男', '女')
);
```

主建的声明可以单独提取出来，使用以下方式声明：

```mysql
# 语法：PRIMARY KEY (列名1, 列名2, ....)

CREATE TABLE student_info (
  number INT,
  name VARCHAR(5),
  gender ENUM('男', '女'),
  PRIMARY KEY (number)
);
```



2、如果主键是多列，必须使用单独声明的方式。

```mysql
CREATE TABLE student_score (
  number INT,
  subject VARCHAR(30),
  score TINYINT,
  PRIMARY KEY (number, subject)
);
```



MySQL会对插入的记录做校验，如果新插入记录的主键值已经在表中存在，则会抛出异常。另外，主键列默认有`NOT NULL`属性，如果填入`NULL`也会报错。



#### UNIQUE属性

对于非主键的其他候选键，如果需要校验值唯一，可以给这个列或列组合添加一个`UNIQUE`属性。

1、如果只给单列设置`UNIQUE`属性，可直接在该列后面填写：

```mysql
CREATE TABLE student_info (
  number INT PRIMARY KEY,
  name VARCHAR(5),
  id_number CHAR(18) UNIQUE
);
```

`UNIQUE`的声明可以单独提取出来，使用以下方式声明：

```mysql
# 语法1：UNIQUE [约束名称] (列名1, 列名2, ...)
# 语法2：UNIQUE KEY [约束名称] (列名1, 列名2, ...)

CREATE TABLE student_info (
  number INT PRIMARY KEY,
  name VARCHAR(5),
  id_nunber CHAR(18),
  UNIQUE KEY uk_id_number (id_number)
);
```



2、如果列组合定义`UNIQUE`属性，必须使用单独声明的方式：

```mysql
CREATE TABLE student_info (
  number INT PRIMARY KEY,
  name VARCHAR(5),
  a_column CHAR(5),
  b_column CHAR(5),
  UNIQUE KEY uk_column (a_column, b_column)
);
```

MySQL会对插入的记录做校验，如果新插入的记录的列或列组合值在表中已经存在，则会抛出异常。

> 约束名称：主键和UNIQUE都属于约束，每个约束都可以有一个名称。主键默认的约束名称是PRIMARY，UNIQUE的约束名称可以自定义或者系统定义。



主键和UNIQUE约束的区别：

- 一张表只能有一个主键，而UNIQUE可以有多个。
- 主键列不允许存NULL，而UNIQUE属性列可以存NULL，且可以重复出现在多条记录中



#### 外键

如果A表的某列或者某些列依赖于B表的某列或者某些列，那么就称A表为子表，B表为父表。子父表可以通过外键关联起来。

```mysql
CONSTRAINT [外键名称] FOREIGN KEY(列1, 列2, ...) REFERENCES 父表名(父列1, 父列2, ...);
```



如，将`student_score`表的`number`定义为外键，依赖于`student_info`表的`number`。

```mysql
CREATE TABLE student_score (
  number INT,
  subject VARCHAR(30),
  score TINYINT,
  PRIMARY KEY (number, subject),
  CONSTRAINT FOREIGN KEY(number) REFERENCES student_info(number)
);
```

当对`student_score`表插入数据时，MySQL会检查插入的`number`是否能在`student_info`表中找到，如果找不到则会抛出异常。



> 父表中被子表依赖的列或列组合必须建立索引，如果该列或者列组合已经是主键或者有UNIQUE属性，那么它们默认被建立了索引。



#### AUTO_INCREMENT（自增长）

如果表中的某列数据类型是整型或浮点型，那么这个列可以设置`AUTO_INCREMENT`属性。自增长列的值是当前列最大值 + 1 后的值。

```mysql
列名 数据类型 AUTO_INCREMENT
```

如，将表`first_table`表中的`id`列设置为主键，并让其自增长。

```mysql
CREATE TABLE first_table (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  first_column INT
);
```



列定义自增长属性时需要注意以下几点：

- 一个表最多有一个具有自增长属性的列。
- 具有自增长属性的列必须建立索引。
- 拥有自增长属性的列不能指定默认值。
- 一般拥有自增长属性的列都是作为主键的列。



#### 列注释

```mysql
列名 数据类型 [列属性] COMMENT '列注释'
```



#### 列的多属性

每个列可以同时具有多个属性，属性声明的顺序没有严格要求，各个属性间用空白隔开即可。有些属性时冲突的，如一个列不能既声明为`PRIMARY KEY`又声明为`UNIQUE KEY`，既声明为`DEFAULT NULL`又声明为`NOT NULL`。