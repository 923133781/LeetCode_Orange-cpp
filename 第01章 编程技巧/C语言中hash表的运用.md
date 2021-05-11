# C语言中哈希表uthash的使用

在Leetcode做题的时候, 发现有人使用哈希表做, 大大降低了时间复杂度, 于是赶快找来学习一下.

C语言中的哈希表是基于开源项目UT_Hash实现的, 在leetcode中已经自动包括其头文件, 因此可以直接使用.

## 定义

头文件

```C++
#include <uthash>
```



```cpp
typedef struct UT_Hash{
UT_hash_handle hh;
type value;//这里是你要存储的数据
int id;//这里是存储的关键字
}UT_Hash
UT_Hash table = NULL, *p1 = NULL, *p2 = NULL;
```

## 查找



```cpp
HASH_FIND_INT(table, &index, p1);
if(p1)
//exist
```

## 增加



```cpp
为p1开辟空间
p1 = (UT_Hash*)malloc(sizeof(UT_Hash));
//add value and index to p1, 填充id和value字段
....
HASH_ADD_INT(table, name, p1) //"name" 是具体的键名, 会在编译的时候被宏替换, 在本文中, name皆为"id"
```

## 删除



```cpp
HASH_DELETE_INT(table, p1);
//这里只是将p1在table中删除了, 并没有free掉整个p1节点
```