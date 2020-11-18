### 一.索引类型

索引可以提升查询速度,会影响where查询,以及order by排序。MySQL索引类型如下：

- 从索引存储结构划分：B Tree索引、Hash索引、FULLTEXT全文索引、R Tree索引；
- 从应用层次划分：普通索引、唯一索引、主键索引、复合索引
- 从索引键值类型划分：主键索引、辅助索引（二级索引）
- 从数据存储和索引键值逻辑关系划分：聚集索引（聚簇索引）、非聚集索引（非聚簇索引）

#### 1.普通索引

**最基本的索引类型，基于普通字段建立的索引，没有任何限制**。创建普通索引方法如下：

- create index <索引的名字> on tablename (fieldname);
- alter table tablename add index [索引的名字] (fieldname);
- create table tablename ([...]，index [索引的名字] (fieldname) );

#### 2.唯一索引

**索引字段值必须唯一，但允许有空值。在创建或修改表时追加唯一约束，会自动创建对应的唯一索引**。

创建唯一索引的方法如下：

- CREATE UNIQUE INDEX <索引的名字> ON tablename (字段名);
- ALTER TABLE tablename ADD UNIQUE INDEX [索引的名字] (字段名);
- CREATE TABLE tablename ( [...], UNIQUE [索引的名字] (字段名) ;

#### 3.主键索引

它是**一种特殊的唯一索引，不允许有空值。在创建或修改表时追加主键约束即可，每个表只能有一个主键**。

创建主键索引的方法如下：

- CREATE TABLE tablename ( [...], PRIMARY KEY (字段名) );
- ALTER TABLE tablename ADD PRIMARY KEY (字段名);

#### 4.复合索引

用户可以**在多个列上建立索引，这种索引叫做组复合索引**（组合索引）。复合索引可以代替多个单一索引，相比多个单一索引复合索引所需的开销更小。

索引同时有两个概念叫做窄索引和宽索引，**窄索引是指索引列为1-2列的索引，宽索引也就是索引列超过2列的索引**。

创建组合索引的方法如下：

- CREATE INDEX <索引的名字> ON tablename (字段名1，字段名2...);
- ALTER TABLE tablename ADD INDEX [索引的名字] (字段名1，字段名2...);
- CREATE TABLE tablename ( [...], INDEX [索引的名字] (字段名1，字段名2...) );

复合索引使用注意事项：

- 何时使用复合索引，要**根据where条件建索引**，注意不要过多使用索引，过多使用会对更新操作效率有很大影响。
- 如果表已经建立了(col1，col2)，就没有必要再单独建立（col1）；如果现在有(col1)索引，如果查
    询需要col1和col2条件，可以**建立(col1,col2)复合索引**，对于查询有一定提高。

#### 5.全文索引

**对于大量的文本数据检索，使用全文索引，查询速度会比like快很多倍**。从MySQL 5.6开始MyISAM和InnoDB存储引擎均支持。创建全文索引的方法如下：

- CREATE FULLTEXT INDEX <索引的名字> ON tablename (字段名);
- ALTER TABLE tablename ADD FULLTEXT [索引的名字] (字段名);
- CREATE TABLE tablename ( [...], FULLTEXT KEY [索引的名字] (字段名) ;

**全文索引有自己的语法格式，使用 `match` 和 `against` 关键字**，比如：

```sql
select * from user
where match(name) against('aaa');
```

**全文索引使用注意事项**：

- 全文索引必须在字符串、文本字段上建立；
- 全文索引字段值必须在最小字符和最大字符之间的才会有效。（innodb：3-84；myisam：4-84）；
- 全文索引字段值要进行切词处理，按syntax字符进行切割，例如b+aaa，切分成b和aaa；
- 全文索引匹配查询，默认使用的是等值匹配，例如a匹配a，不会匹配ab,ac。如果想匹配可以在布
    尔模式下搜索a*。

```sql
select * from user
where match(name) against('a*' in boolean mode);
```

### 二.索引原理

#### 1.二分查找法





#### 2.Hash结构

```
show engine innodb status \G;
show variables like '%innodb_adaptive%';
set global innodb_adaptive_hash_index=0; // 关闭自适应hash索引; 1为开启
```





#### 3.B+Tree结构





#### 4.聚簇索引和辅助索引





### 三.索引分析与优化



### 四.查询优化





添加数据：

insert into slow(name) select name from slow;