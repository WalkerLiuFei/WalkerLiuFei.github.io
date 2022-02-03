# Redis 中的数据结构

## 动态字符串（SDS）

### SDS 数据结构

```c
struct sdshdr {

    // buf 已占用长度
    int len;

    // buf 剩余可用长度
    int free;

    // 实际保存字符串数据的地方
    char buf[];
};
```



### SDS 的特性

1. 二进制安全 ： C字符串的数据肯定满足某种编码的，而SDS不需要。
2. 获取字符串长度 的时间复杂度为 $$O(1)$$，因为有一个专门的记录长度的属性，如果使用使用原始的`C`字符串，计算长度的复杂度为$$O(N)$$。
3. 避免缓存区溢出 ： 比如两个字符串相 concat，C字符串如果目标字符串没有提前分配足够的空间，这个操作就会造成缓冲区溢出，而SDS的API封装了这些检查和分配策略。
4. 内存开销小，对于经常变化的字符串，有预分配的策略
   1. < 1MB ，预分配内存大小为需要内存的两倍
   2. /> 1MB , 多分配1MB空间
   3. 预分配的内存，在字符串被删除之前，不会被删除
   4. **这里需要注意的点是，对于经常append 的字符串，需要及时释放多余的预分配内存**



## 双端链表

Redis通过 双端链表和压缩列表实现的集合功能

### 数据结构

```c
typedef struct list {

    // 表头指针
    listNode *head;

    // 表尾指针
    listNode *tail;

    // 节点数量
    unsigned long len;

    // 复制函数
    void *(*dup)(void *ptr);
    // 释放函数
    void (*free)(void *ptr);
    // 比对函数
    int (*match)(void *ptr, void *key);
} list;
```

```c
typedef struct listNode {

    // 前驱节点
    struct listNode *prev;

    // 后继节点
    struct listNode *next;

    // 值
    void *value;

} listNode;
```





Redis 有两种实现 **双端链表**，**压缩列表**。 **Redis在创建集合时默认使用压缩列表的数据结构，只有在不得不适用双端链表时在使用双端链表。**

- 事务模块使用双端链表依序保存输入的命令；
- 服务器模块使用双端链表来保存多个客户端；
- 订阅/发送模块使用双端链表来保存订阅模式的多个客户端；
- 事件模块使用双端链表来保存时间事件（time event）；

### 相关的操作

列表的先关操作底层都是通过链表实现的。

XPUSH ， XPOP，等等