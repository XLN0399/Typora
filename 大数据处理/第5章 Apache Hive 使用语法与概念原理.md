# 数据库操作



创建数据库

```hive
create database [if not exists] myshive;
```

使用数据库

```hive
use myhive;
```

查看数据库详情

```hive
des database myhive;
```



数据库本质就是HDFS之上的文件夹

默认数据库存放路径是HDFS的`/user/hive/warehouse`内



创建数据库可指定HDFS中存放路径

```hive
create database myhive2 location '/myhive2';
```

使用`location`关键字，可以指定数据库在HDFS的存储路径



删除数据库

```hive
drop database myhive2 [cascade];
```

如果数据库中有表，删除会报错，使用关键字`cascade`，强制删除数据库，包含数据库下面的表



```hive
create database [if not exists] db_name [location file_path];

drop database db_name [cascade]
```





# 数据表操作



## 表操作语法和数据类型



### 数据类型

| **分类** | **类型**  | **描述**                                       | **字面量示例**                                               |
| -------- | --------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 原始类型 | BOOLEAN   | true/false                                     | TRUE                                                         |
|          | TINYINT   | 1字节的有符号整数 -128~127                     | 1Y                                                           |
|          | SMALLINT  | 2个字节的有符号整数，-32768~32767              | 1S                                                           |
|          | INT       | 4个字节的带符号整数                            | 1                                                            |
|          | BIGINT    | 8字节带符号整数                                | 1L                                                           |
|          | FLOAT     | 4字节单精度浮点数1.0                           |                                                              |
|          | DOUBLE    | 8字节双精度浮点数                              | 1.0                                                          |
|          | DEICIMAL  | 任意精度的带符号小数                           | 1.0                                                          |
|          | STRING    | 字符串，变长                                   | “a”,’b’                                                      |
|          | VARCHAR   | 变长字符串                                     | “a”,’b’                                                      |
|          | CHAR      | 固定长度字符串                                 | “a”,’b’                                                      |
|          | BINARY    | 字节数组                                       |                                                              |
|          | TIMESTAMP | 时间戳，毫秒值精度                             | 122327493795                                                 |
|          | DATE      | 日期                                           | ‘2016-03-29’                                                 |
|          |           | 时间频率间隔                                   |                                                              |
| 复杂类型 | ARRAY     | 有序的的同类型的集合                           | array(1,2)                                                   |
|          | MAP       | key-value,key必须为原始类型，value可以任意类型 | map(‘a’,1,’b’,2)                                             |
|          | STRUCT    | 字段集合,类型可以不同                          | struct(‘1’,1,1.0), named_stract(‘col1’,’1’,’col2’,1,’clo3’,1.0) |
|          | UNION     | 在有限取值范围内的一个值                       | create_union(1,’a’,63)                                       |



### 基础建表

```sql
CREATE [EXTERNAL] TABLE tb_name
	(col_name col_type [COMMENT col_comment], ......)
	[COMMENT tb_comment]
	[PARTITIONED BY(col_name, col_type, ......)]
	[CLUSTERED BY(col_name, col_type, ......) INTO num BUCKETS]
	[ROW FORMAT DELIMITED FIELDS TERMINATED BY '']
	[LOCATION 'path']
```

- `[EXTERNAL]`，外部表，需搭配

  - `[ROW FORMAT DELIMITED FIELDS TERMINATED BY '']`指定列分隔符

  - `[LOCATION 'path']`表数据路径

  - 外部表示意

    ```sql
    CREATE EXTERNAL TABLE test_ext(id int) COMMENT 'external table' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LOCATION 'hdfs://node1:8020/tmp/test_ext';
    ```

    

- `[COMMENT tb_comment]`表注释，可选

- `[PARTITIONED BY(col_name, col_type, ......)]`基于列分区

  ```sql
  -- 分区表示意
  CREATE TABLE test_ext(id int) COMMENT 'partitioned table' PARTITION BY(year string, month string, day string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
  ```

- `[CLUSTERED BY(col_name, col_type, ......)]`基于列分桶

  ```sql
  CREATE TABLE course (c_id string,c_name string,t_id string) CLUSTERED BY(c_id) INTO 3 BUCKETS ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';
  ```





<hr>

Hive中创建表可分为内部表，外部表，分区表，分桶表，不同类型的表有各自的用途。

- 内部表

```hive
create table table_name ...
```

未被`external`关键字修饰即为内部表，即普通表。内部表又称为管理表，内部表数据存储的位置由`hive.metastore.warehouse.dir`参数决定，默认（`/user/hive/warehouse`），**删除内部表会直接删除元数据（metadata）及存储数据**，因此内部表不适合和其他工具共享数据。



- 外部表

```hive
create external table table_name ... location ...
```

被`external`关键字修饰即是外部表，即关联表。外部表是指表数据可以在任何位置，通过`location`关键字指定。数据存储的不同也代表这个表在理念是并不是被Hive内部管理的，而是可以随意临时链接到外部数据上的。



在删除外部表的时候，仅仅是删除元数据（表的信息），不会删除数据本身。



## 内部表操作



代码示例

```hive
create database if not exists myhive;
use myhive;
create table if not exists stu(id int, name string);
insert into stu values (1, '馨雅'), (2, '南笙');
select * from stu;
```



结果示例

在HDFS上。查看表的数据存储文件

![image-20240301140835831](assets/image-20240301140835831.png)





从显示结果，发现两列数据并没有分隔，这是应为默认的分隔符是`\001`，是一种特殊字符，ASCII值，键盘是打不出来的，在有些文本编辑器上显示`SOH`

分隔符可以自行指定，在创建表的时候指定

```hive
create table if not exists stu2(id int, name string) row format delimited fields terminated by '\t';
insert into stu2 values (1, '馨雅'), (2, '南笙');
```

这里设置为`\t`为分隔符，查看新创建的文件

![image-20240301141440887](assets/image-20240301141440887.png)





其他创建表的形式

- 基于查询结果建表

```hive
create table stu3 as select * from stu2;
```

![image-20240301141836766](assets/image-20240301141836766.png)

在建表时，只要没有指定间隔符，都是以默认`\001`作为间隔符



- 基于已存在的表结构建表

```hive
create table stu4 like stu2;
```

这个建表只是创建表结构，表中的内容，并不会被创建出。且这个间隔符是是会根据原来表的设定而设置



查看表类型和详情

```hive
desc formatted stu2;
```



删除内部表

```hive
drop table table_name;
```

**删除内部表，表信息以及表数据全部都被删除**





## 外部表操作

外部表，创建表被`external`修饰，从概念上是被认为并非Hive拥有的表，只是临时关联数据去使用

外部表和数据是相互独立，即

- 可以先有表，然后把数据移动到表指定的location中
- 也可以先有数据，然后创建表通过location执行数据





1. 创建外部表

```hive
create external table test_external_table1(id int, name string) 
row format delimited fields terminated by '\t' location '/tmp/test_ext1';
```

2. 创建完表，可以查看表中并没有数据

```hive
select * from test_external_table1;
```

3. 在Linux本地创建文件，然后移动到HDFS中

```sh
hdfs dfs -put test_data.txt /tmp/test_ext1
```

示例文件数据`test_data.txt`，数据列用`\t`分隔

```tex
1	馨雅
2	南笙
```

4. 直接查看结果

```hive
select * from test_external_table1;
```



1. 先存在数据，后创建表。数据沿用上一次上传了的数据
2. 创建表

```hive
create external table test_ext2(id int, name string) row format delimited fields terminated by '\t' location '/tmp/test_ext1';
```

3. 查看数据

```hive
select * from test_ext2;
```



从这两次操作，两张表用的是同一个数据。数据和表之间是映射关系，两张表映射到同一个数据上

删除外部表和内部表操作相同

```hive
drop table test_ext2;
```

这里的删除只会删除表的元数据，不会真正去删除数据文件



内外部表转换

- 查看表类型

```hive
desc formatted table_name;
```

- 内部表转外部表

```hive
alter table stu set tblproperties ('EXTERNAL'='TRUE');
```

- 外部表转内部表

```hive
alter table test_ext2 set tblproperties ('EXTERNAL'='FALSE');
```

`'EXTERNAL'='FALSE'`，一定是大写



## 数据加载和导出



### LOAD语法 加载



```hive
load data [local] inpath 'filepath' [overwrite] into table table_name;
```

- `load data`：加载数据
- `local`：是否在Linux本地还是HDFS文件系统中
- `'filepath'`：数据文件路径
- `overwrite`：是否覆盖或者追加



**如果基于HDFS进行load加载数据，源数据文件会消失，本质是移动到表所在目录中**



### INSERT SELECT语法 加载



```hive
insert [overwrite|into] table table_name [partition (partcol1=val1, partcol2=val2, ...) [if not exists]] select select_statement from from_statememt;
```

将select查询语句的结果插入到其他表中，被select查询的表可以是内部表或外部表



数据在本地推荐 `load data`

数据在HDFS

- 如果不保留原始文件，推荐使用`load data`
- 如果保留原始文件，推荐使用`insert select`

数据已经在表中

- 只可以`insert select`



### INSERT OVERWRITE语法 导出



```hive
insert overwrite [local] directory 'file_dir' [row format delimited fields terminated by '\t'] select select_statement from from_statement []
```

在不指定列分隔符，使用默认列分隔符，`local`选择是导出到Linux本地还是HDFS文件系统中



### hive shell 导出

通过`/export/server/hive/bin/hive`导出

```sh
/export/server/hive/bin/hive -e "sql语句" > file_path
/export/server/hive/bin/hive -f "sql脚本路径" > file_path
```

这里`file_path`指定到具体文件名