<style>
table {
    width: 100%; /*表格宽度*/
    max-width: 65em; /*表格最大宽度，避免表格过宽*/
    border: 1px solid #dedede; /*表格外边框设置*/
    margin: 15px auto; /*外边距*/
    border-collapse: collapse; /*使用单一线条的边框*/
    empty-cells: show; /*单元格无内容依旧绘制边框*/
}
table th,
table td {
  height: 35px; /*统一每一行的默认高度*/
  border: 1px solid #dedede; /*内部边框样式*/
  padding: 0 10px; /*内边距*/
}
table th {
    font-weight: bold; /*加粗*/
    text-align: center !important; /*内容居中，加上 !important 避免被 Markdown 样式覆盖*/
    background: rgba(158,188,226,0.2); /*背景色*/
}
table tbody tr:nth-child(2n) {
    background: rgba(158,188,226,0.12); 
}
table tr:hover {
    background: #efefef; 
}
table th {
    white-space: nowrap; /*表头内容强制在一行显示*/
}
</style>
### 概述

>这里只是讲解一下MySQL中常用的两种存储引擎,InnoDB和MyISAM。

>MyISAM是MySQL的默认数据库引擎（5.5版之前），由早期的ISAM（Indexed Sequential Access Method：有索引的顺序访问方法）所改良。虽然性能极佳，但却有一个缺点：不支持事务处理（transaction）。


### 两者之间的区别

| 特性 |MyISAM   |  InnoDB |
|--| --| --|
|事务支持  |MyISAM不支持事务, InnoDB的AUTOCOMMIT默认是打开的，即每条SQL语句会默认被封装成一个事务，自动提交，这样会影响速度，所以最好是把多条SQL语句显示放在begin和commit之间，组成一个事务去提交。<br/> MyISAM是非事务安全型的，默认开启自动提交，宜合并事务，一同提交，减小数据库多次提交导致的开销，大大提高性能。| InnoDB支持事务，而且是事务安全的
| 存储结构 | MyISAM:每个MyISAM在磁盘上存储成三个文件。第一个文件的名字以表的名字开始，扩展名指出文件类型。.frm文件存储表定义。数据文件的扩展名为.MYD (MYData)。索引文件的扩展名是.MYI (MYIndex)。  | InnoDB：所有的表都保存在同一个数据文件中（也可能是多个文件，或者是独立的表空间文件），InnoDB表的大小只受限于操作系统文件的大小，一般为2GB。|
| 存储空间 |MyISAM：可被压缩，存储空间较小。支持三种不同的存储格式：静态表(默认，但是注意数据末尾不能有空格，会被去掉)、动态表、压缩表。   | InnoDB：需要更多的内存和存储，它会在主内存中建立其专用的缓冲池用于高速缓冲数据和索引。  |
| 可移植性、备份和恢复 |MyISAM：数据是以文件的形式存储，所以在跨平台的数据转移中会很方便。在备份和恢复时可单独针对某个表进行操作。   |InnoDB：免费的方案可以是拷贝数据文件、备份 binlog，或者用 mysqldump，在数据量达到几十G的时候就相对痛苦了。   |
|事务支持  |   可以和其他字段一起建立联合索引。引擎的自动增长列必须是索引，如果是组合索引，自动增长可以不是第一列，他可以根据前面几列进行排序后递增。|   InnoDB中必须包含只有该字段的索引。引擎的自动增长列必须是索引，如果是组合索引也必须是组合索引的第一列。|
| 锁表差异 |只支持表级锁，用户在操作myisam表时，select，update，delete，insert语句都会给表自动加锁，如果加锁以后的表满足insert并发的情况下，可以在表的尾部插入新的数据。<br>MyISAM同一个表上的读锁和写锁是互斥的，MyISAM并发读写时如果等待队列中既有读请求又有写请求，默认写请求的优先级高，即使读请求先到，所以MyISAM不适合于有大量查询和修改并存的情况，那样查询进程会长时间阻塞。因为MyISAM是锁表，所以某项读操作比较耗时会使其他写进程饿死。| 支持事务和行级锁，是innodb的最大特色。行锁大幅度提高了多用户并发操作的新能。但是InnoDB的行锁，只是在WHERE的主键是有效的，非主键的WHERE都会锁全表的。  |
|全文索引  |支持(FULLTEXT类型的)全文索引   |  不支持(FULLTEXT类型的)全文索引，但是innodb可以使用sphinx插件支持全文索引，并且效果更好。 |
| 表主键 |允许没有任何索引和主键的表存在,索引都是保存行的地址   |  如果没有设定主键或者非空唯一索引，就会自动生成一个6字节的主键(用户不可见)，数据是主索引的一部分，附加索引保存的是主索引的值。InnoDB的主键范围更大，最大是MyISAM的2倍。 |
| 表的具体行数 |  保存有表的总行数，如果select count(*) from table;会直接取出出该值。 | 没有保存表的总行数(只能遍历)，如果使用select count(*) from table；就会遍历整个表，消耗相当大，但是在加了wehre条件后，myisam和innodb处理的方式都一样。  |
|  CURD操作| 如果执行大量的SELECT，MyISAM是更好的选择。  | 如果你的数据执行大量的INSERT或UPDATE，出于性能方面的考虑，应该使用InnoDB表。DELETE 从性能上InnoDB更优，但DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的删除，在innodb上如果要清空保存有大量数据的表，最好使用truncate table这个命令。  |
|外键  | 不支持  |支持   |
|查询效率  |   。如果应用中需要执行大量的SELECT查询，那么MyISAM是更好的选择| 如果应用中需要执行大量的INSERT或UPDATE操作，则应该使用InnoDB，这样可以提高多用户并发操作的性能。  |
