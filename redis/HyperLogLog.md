# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;HyperLogLog是一种基数算法,通过HyperLogLog可以利用很小的内存空间完成独立总数的统计。命令如下所示:

```redis
pfadd key element[element]


pfcount kye[key]

pfmerge [key][key]

注意:HyperLogLog统计总数的时候可能存在误差。
```