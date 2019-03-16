### B+树的性质

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/9.jpg?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/10.jpg?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/11.jpg?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/12.jpg?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/13.jpg?raw=true)


### 图解B+树
>&nbsp;&nbsp;&nbsp;&nbsp; 原博客地址:https://blog.csdn.net/qq_26222859/article/details/80631121

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/14.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/15.png?raw=true)


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/20.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/21.png?raw=true)


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/22.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/23.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/24.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/25.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/26.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/27.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/28.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/29.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/30.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/31.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/32.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/33.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/34.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/35.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;需要补充的是，在数据库的聚集索引（Clustered Index）中，叶子节点直接包含卫星数据。在非聚集索引（NonClustered Index）中，叶子节点带有指向卫星数据的指针。
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/36.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/37.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/38.png?raw=true)
>&nbsp;&nbsp;&nbsp;&nbsp;第一次磁盘IO
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/39.png?raw=true)
>&nbsp;&nbsp;&nbsp;&nbsp;第二次磁盘IO
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/40.png?raw=true)
>&nbsp;&nbsp;&nbsp;&nbsp;第三次磁盘IO
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/41.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/42.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/43.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/44.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/45.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/46.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/47.png?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;自顶向下，查找到范围的下限（3）：
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/48.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;中序遍历到元素6：
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/49.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;中序遍历到元素8：
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/50.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;中序遍历到元素9：
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/51.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;中序遍历到元素11：
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/52.png?raw=true)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/53.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/54.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;自顶向下，查找到范围的下限（3）：
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/55.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;通过链表指针，遍历到元素6, 8：
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/56.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;通过链表指针，遍历到元素9, 11，遍历结束：

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/57.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/58.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/59.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/60.png?raw=true)
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-16/61.png?raw=true)

```text
B+树的特征：

1.有k个子树的中间节点包含有k个元素（B树中是k-1个元素），每个元素不保存数据，只用来索引，所有数据都保存在叶子节点。

2.所有的叶子结点中包含了全部元素的信息，及指向含这些元素记录的指针，且叶子结点本身依关键字的大小自小而大顺序链接。

3.所有的中间节点元素都同时存在于子节点，在子节点元素中是最大（或最小）元素。

B+树的优势：

1.单一节点存储更多的元素，使得查询的IO次数更少。

2.所有查询都要查找到叶子节点，查询性能稳定。

3.所有叶子节点形成有序链表，便于范围查询
```