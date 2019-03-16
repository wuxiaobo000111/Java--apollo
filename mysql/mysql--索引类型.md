### 概述

>&nbsp;&nbsp;&nbsp;&nbsp;这里讲述一下mysql中的索引类型，以及常见的InnoDB和MyIASM引擎中可以使用的索引。

### 索引类型

#### 主键索引

>&nbsp;&nbsp;&nbsp;&nbsp;主键是一种唯一性索引，但它必须指定为“PRIMARY KEY”。如果你曾经用过AUTO_INCREMENT类型的列，你可能已经熟悉主键之类的概念了。主键一般在创建表的时候指定，例如“CREATE TABLE tablename ( [...], PRIMARY KEY (列的列表) ); ”。但是，我们也可以通过修改表的方式加入主键，例如“ALTER TABLE tablename ADD PRIMARY KEY (列的列表); ”。每个表只能有一个主键。

#### 唯一索引
>&nbsp;&nbsp;&nbsp;&nbsp;唯一索引，即是唯一的意思，在数据库表结构中对字段添加唯一索引后进行数据库进行存储操作时数据库会判断库中是否已经存在此数据，不存在此数据时才能进行插入操作。

##### 唯一索引与唯一约束的区别

>&nbsp;&nbsp;&nbsp;&nbsp;在mysql中貌似唯一约束就是唯一索引，并没有什么不同，可能叫法不同，在sqlserver中区分还是挺明确的。 

>&nbsp;&nbsp;&nbsp;&nbsp;博客中的一句话说的很在理，你为了做到数据不能有重复值，但是数据库怎么保证没有重复值呢？当然是在存储数据的时候查一遍，那么怎样查找快呢？ 当然是创建索引，所以，在创建唯一约束的时候就创建了唯一索引。这可能也是mysql的一个优化机制。

```mysql
添加索引
alter table table_name add unique(column)
删除索引
alter table table_name drop index colum_name
```

>&nbsp;&nbsp;&nbsp;&nbsp;注意:唯一索引中的值可以有一个为null，但是主键索引并不能为null。

#### 普通索引

```mysql
第一种方式 :

  CREATE INDEX account_Index ON `award`(`account`);
第二种方式: 

ALTER TABLE award ADD INDEX account_Index(`account`)
```


#### 复合索引


#### 全文索引

>&nbsp;&nbsp;&nbsp;&nbsp;即为全文索引，目前只有MyISAM引擎支持。其可以在CREATE TABLE ，ALTER TABLE ，CREATE INDEX 使用，不过目前只有 CHAR、VARCHAR ，TEXT 列上可以创建全文索引。


#### 聚簇索引

>&nbsp;&nbsp;&nbsp;&nbsp;Innodb的聚簇索引在同一个B-Tree中保存了索引列和具体的数据，在聚簇索引中，实际的数据保存在叶子页中，中间的节点页保存指向下一层页面的指针。“聚簇”的意思是数据行被按照一定顺序一个个紧密地排列在一起存储。一个表只能有一个聚簇索引，因为在一个表中数据的存放方式只有一种。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190315143608702.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI1NDg0MTQ3,size_16,color_FFFFFF,t_70)

>&nbsp;&nbsp;&nbsp;&nbsp;这是个B+ 树的结构,关于B+ 树会在之后的内容中讲到。这里只是说明在InnoDB中,索引是使用B+ 树的结构，在这个结构中，只有叶子节点才会存在节点数组，其他的只是会存放索引值。

##### 聚簇索引的优缺点

>优点:<br/>&nbsp;&nbsp;&nbsp;&nbsp;聚簇索引将索引和数据行保存在同一个B-Tree中，查询通过聚簇索引可以直接获取数据，相比非聚簇索引需要第二次查询（非覆盖索引的情况下）效率要高。
&nbsp;&nbsp;&nbsp;&nbsp;聚簇索引对于范围查询的效率很高，因为其数据是按照大小排列的，


>缺点:
&nbsp;&nbsp;&nbsp;&nbsp;1.聚簇索引的更新代价比较高，如果更新了行的聚簇索引列，就需要将数据移动到相应的位置。这可能因为要插入的页已满而导致“页分裂”。
&nbsp;&nbsp;&nbsp;&nbsp;2.插入速度严重依赖于插入顺序，按照主键进行插入的速度是加载数据到Innodb中的最快方式。如果不是按照主键插入，最好在加载完成后使用OPTIMIZE TABLE命令重新组织一下表。
&nbsp;&nbsp;&nbsp;&nbsp;3.聚簇索引在插入新行和更新主键时，可能导致“页分裂”问题。
&nbsp;&nbsp;&nbsp;&nbsp;4.聚簇索引可能导致全表扫描速度变慢，因为可能需要加载物理上相隔较远的页到内存中（需要耗时的磁盘寻道操作）。


#### 非聚簇索引

>&nbsp;&nbsp;&nbsp;&nbsp;非聚簇索引，又叫二级索引。二级索引的叶子节点中保存的不是指向行的物理指针，而是行的主键值。当通过二级索引查找行，存储引擎需要在二级索引中找到相应的叶子节点，获得行的主键值，然后使用主键去聚簇索引中查找数据行，这需要两次B-Tree查找。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190315145432234.)



#### 聚簇索引和非聚簇索引的使用场景



#### InnoDB使用的索引结构(B+树,注意:是叫B树)
[B-树](https://github.com/wuxiaobo000111/markdown/blob/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/B%E6%A0%91.md "B-树")

[B+树](https://github.com/wuxiaobo000111/markdown/blob/master/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/B%2B%E6%A0%91.md "B+树")

>&nbsp;&nbsp;&nbsp;&nbsp;B-树和B+树的原理都在如上的两篇文章中。


#### mysql索引失效

> &nbsp;&nbsp;&nbsp;&nbsp;不想重复写了，这里献上我的一个笔记的链接地址:http://note.youdao.com/noteshare?id=5b32c54e21d5af4f1107665e173a7989&sub=D1FBD8FA38564234A049A48D8A1BAD12