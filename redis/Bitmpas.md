# Bitmaps简介

>&nbsp;&nbsp;&nbsp;&nbsp;redis为Bitmaps提供了一套单独的命令。可以想象为Bitmaps是一个已位为单位的数组,数组中的每个元素只能存储0和1,数组中的下标在Bitmaps叫偏移量。其实本身还是用字符串类进行存储。

# 命令

```redis
setbit key offset value

    设置键的第offset个位上的值。

getbit key offset

    获取键的第offset的值

bitcount key  [start][end]

    获取Bitmaps指定范围值为1的个数(start和end表示起始和结束字节数)

bitop  运算符  destkey key[key......]

    可以做多个Bitmaps的and(交集),or(并集),not(非),xor(异或)然后将结果保存在destkey中。

bitpos key targetBit [start][end]


    计算Bitmaps中的第一个值为targetBit的偏移量。
```