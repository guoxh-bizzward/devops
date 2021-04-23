# Redis实战

## SDS

`Simple Dynamic String` 简单动态字符串

`sds.h` `sds.c`

## 链表

Redis自己实现的链表

**列表的底层实现之一就是链表.**除了链表之外,发布与订阅,慢查询,监视器等功能也用到了链表;Redis服务器本身也使用了链表来保存多个客户端的状态信息,以及使用链表来构建客户端输出缓冲区.

`alist.h` 

```
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;
```

```
typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```

Redis的链表有以下特性

* **双端** 链表节点带有 pre 和 next指针,获取节点的前置节点和后置节点的时间复杂度都是O(1)
* **无环** 链表的表头节点和表尾节点
* **带表头指针和表尾指针** 通过list接口的head指针和tail指针,程序获取链表的表头节点和表尾节点的时间复杂度为O(1)
* **带链表长度计数器** 自带len属性,获取链表节点长度的时间复杂度为O(1)
* **多态** 链表节点使用void指针来保存节点值,并且可以通过list结构的dup,free,match三个属性为节点值设置类型特定函数,所以链表可以用来保存不同类型的值.

## 字典

保存键值对的抽象数据结构

Redis的数据库就是使用字典表作为底层实现的,对数据库的增删改查操作也是构建在字典的操作之上的.

字典表是哈希键的底层实现之一.当一个哈希键包含的键值对比较多,又或者键值对中的元素都是比较长的字符串时,redis就会使用字典作为哈希键的底层实现.

> 这段的描述是说的hash key对应的value,而不是 redis 数据结构中的hash 

`dict.h`

```
typedef struct dictht {
    dictEntry **table;//哈希表的数组
    unsigned long size; //哈希表的大小
    unsigned long sizemask; //哈希表大小掩码,用于计算索引值,总是等于 size-1
    unsigned long used;//该哈希表已有节点的数量
} dictht;

typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;
```

> sizemak 和 hash值一汽决定一个键应该被放到table数组的哪个索引上面;