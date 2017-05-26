# nginx基本数据类型  
## 整形数据  
- 有符号整数 ngx_int_t
- 无符号整数 ngx_uint_t
- 开关类型，ngx_flag_t
> 实际定义与ngx_int_t一致，用于表示某些选项的开关.例如 
		multi_accept on		对应ngx_flag_t变量的值为1   
		multi_accept off		对应ngx_flag_t变量的值为0

## 字符串类型  
> nginx对字符串进行了自己的封装，特点是通过指针和长度来表示，**没有使用‘\0’作为字符串的结束符**。
> 定义如下：
```c
/*
src/core/ngx_string.h
*/
/* ngx_str_t 在 u_char 的基础上增加了字符串长度的信息，即len变量 */
typedef struct {
    size_t      len;    /* 字符串的长度 */
    u_char     *data;   /* 指向字符串的第一个字符 */
} ngx_str_t;

#define ngx_string(str) {sizeof(str)-1, (u_char *) str}
#define ngx_null_string {0, NULL}
#define ngx_str_set(str, text)
    (str)->len = sizeof(text)-1; (str)->data = (u_char *)text

#define ngx_str_null(str)   (str)->len = 0; (str)->data = NULL

```
>ngx字符串的初始化方式：
```c

/* 正确写法*/
ngx_str_t str1 = ngx_string("hello nginx");
ngx_str_t str2 = ngx_null_string;

/* 错误写法*/
ngx_str_t str1, str2;
str1 = ngx_string("hello nginx");   /* 编译出错 */
str2 = ngx_null_string;             /* 编译出错 */

/* 正确写法*/
ngx_str_t str1, str2;
ngx_str_set(&str1, "hello nginx");
ngx_str_null(&str2);

```

# ngx基本数据结构  
## ngx_keyval_t  
用于存储key-value对，比如各种http header的名字和值。
定义为
```c
/*
src/core/ngx_string.h
*/
typedef struct {
    ngx_str_t   key;
    ngx_str_t   value;
} ngx_keyval_t;

```
## ngx_array_t
ngx_array_t 是一个顺序容器，它在 Nginx 中大量使用。 ngx_array_t 容器以数组的形式存储元素，并支持在达到数组容量的上限时**动态**改变数组的大小。
定义如下：
```c
/*
src/core/ngx_array.h
*/
typedef struct {
    void        *elts;  /* 指向数组数据区域的首地址 */
    ngx_uint_t   nelts; /* 数组实际元素的个数 */
    size_t       size;  /* 单个元素所占据的字节大小 */
    ngx_uint_t   nalloc;/* 数组容量，最多存储元素个数 */
    ngx_pool_t  *pool;  /* 数组对象所在的内存池， 数组数据区域所需的空间都由这个pool来提供*/
} ngx_array_t;
```
动态数组支持的基本操作:
```c
/*
src/core/ngx_array.c
*/
/* 创建新的动态数组 */
ngx_array_t *ngx_array_create(ngx_pool_t *p, ngx_uint_t n, size_t size);
/* 销毁数组对象，内存被内存池回收 */
void ngx_array_destroy(ngx_array_t *a);
/* 在现有数组中增加一个新的元素 */
void *ngx_array_push(ngx_array_t *a);
/* 在现有数组中增加 n 个新的元素 */
void *ngx_array_push_n(ngx_array_t *a, ngx_uint_t n);
```
## ngx_list_t
ngx_list_t 是 Nginx 封装的单向链表容器。
Nginx 链表容器和普通链表类似，均有链表表头和链表节点，通过节点指针组成链表。其结构定义如下：
```c
/*
src/core/ngx_list.h
*/
/* 链表结构 */
typedef struct ngx_list_part_s  ngx_list_part_t;

/* 链表中的节点结构 */
struct ngx_list_part_s {
    void             *elts; /* 指向该节点数据区的首地址 */
    ngx_uint_t        nelts;/* 该节点数据区实际存放的元素个数 */
    ngx_list_part_t  *next; /* 指向链表的下一个节点 */
};

/* 链表表头结构 */
typedef struct {
    ngx_list_part_t  *last; /* 指向链表中最后一个节点 */
    ngx_list_part_t   part; /* 链表中表头包含的第一个节点 */
    size_t            size; /* 元素的字节大小 */
    ngx_uint_t        nalloc;/* 链表中每个节点所能容纳元素的个数 */
    ngx_pool_t       *pool; /* 该链表节点空间的内存池对象 */
} ngx_list_t;
```
链表支持的基本操作只有两个：
```c
/* 创建链表 */
ngx_list_t * ngx_list_create(ngx_pool_t *pool, ngx_uint_t n, size_t size);

/* 添加一个元素 */
void * ngx_list_push(ngx_list_t *l);
```
由于链表的内存分配是基于内存池，所有内存的销毁由内存池进行，即链表没有销毁操作。
## ngx_queue_t
nginx的双向循环链表结构。在 Nginx 的队列实现中，实质就是具有头节点的双向循环链表，这里的双向链表中的节点是没有数据区的，只有两个指向节点的指针。需注意的是队列链表的内存分配不是直接从内存池分配的，即没有进行内存池管理，而是需要我们自己管理内存，所有我们可以指定它在内存池管理或者直接在堆里面进行管理，最好使用内存池进行管理。节点结构定义如下：

```c
/* 队列结构，其实质是具有有头节点的双向循环链表 */
typedef struct ngx_queue_s  ngx_queue_t;

/* 队列中每个节点结构，只有两个指针，并没有数据区 */
struct ngx_queue_s {
    ngx_queue_t  *prev;
    ngx_queue_t  *next;
};
```
基本操作如下：
```c
/* h 为链表结构体 ngx_queue_t 的指针；初始化双链表 */
ngx_queue_int(h)

/* h 为链表容器结构体 ngx_queue_t 的指针； 判断链表是否为空 */
ngx_queue_empty(h)

/* h 为链表容器结构体 ngx_queue_t 的指针，x 为插入元素结构体中 ngx_queue_t 成员的指针；将 x 插入到链表头部 */
ngx_queue_insert_head(h, x)

/* h 为链表容器结构体 ngx_queue_t 的指针，x 为插入元素结构体中 ngx_queue_t 成员的指针。将 x 插入到链表尾部 */
ngx_queue_insert_tail(h, x)

/* h 为链表容器结构体 ngx_queue_t 的指针。返回链表容器 h 中的第一个元素的 ngx_queue_t 结构体指针 */
ngx_queue_head(h)

/* h 为链表容器结构体 ngx_queue_t 的指针。返回链表容器 h 中的最后一个元素的 ngx_queue_t 结构体指针 */
ngx_queue_last(h)

/* h 为链表容器结构体 ngx_queue_t 的指针。返回链表结构体的指针 */
ngx_queue_sentinel(h)

/* x 为链表容器结构体 ngx_queue_t 的指针。从容器中移除 x 元素 */
ngx_queue_remove(x)

/* h 为链表容器结构体 ngx_queue_t 的指针。该函数用于拆分链表，
 * h 是链表容器，而 q 是链表 h 中的一个元素。
 * 将链表 h 以元素 q 为界拆分成两个链表 h 和 n
 */
ngx_queue_split(h, q, n)

/* h 为链表容器结构体 ngx_queue_t 的指针， n为另一个链表容器结构体 ngx_queue_t 的指针
 * 合并链表，将 n 链表添加到 h 链表的末尾
 */
ngx_queue_add(h, n)

/* h 为链表容器结构体 ngx_queue_t 的指针。返回链表中心元素，即第 N/2 + 1 个 */
ngx_queue_middle(h)

/* h 为链表容器结构体 ngx_queue_t 的指针，cmpfunc 是比较回调函数。使用插入排序对链表进行排序 */
ngx_queue_sort(h, cmpfunc)

/* q 为链表中某一个元素结构体的 ngx_queue_t 成员的指针。返回 q 元素的下一个元素。*/
ngx_queue_next(q)

/* q 为链表中某一个元素结构体的 ngx_queue_t 成员的指针。返回 q 元素的上一个元素。*/
ngx_queue_prev(q)

/* q 为链表中某一个元素结构体的 ngx_queue_t 成员的指针，type 是链表元素的结构体类型名称，
 * link 是上面这个结构体中 ngx_queue_t 类型的成员名字。返回 q 元素所属结构体的地址
 */
ngx_queue_data(q, type, link)

/* q 为链表中某一个元素结构体的 ngx_queue_t 成员的指针，x 为插入元素结构体中 ngx_queue_t 成员的指针 */
ngx_queue_insert_after(q, x)
```

## ngx_buf_t
nginx在处理数据时，大量使用ngx_buf_t数据结构，用来保存从网络上收到的数据或在要发送到网络上的数据。
在不同的场景下，ngx_buf_t中的各个指针代表着不同的意义。
比如，经典的生产者/消费者场景，同时操作一个ngx_buf_t时：
	start-pos 区间：消费者已经处理的区间
	pos-last 区间：生产者已经写入，尚未被消费者处理的数据区间
	last-end区间：生产者可以写入的区间	
nginx在使用数据时，为了提高效率，将尽可能的使用指针、偏移位置，避免进行内存拷贝。在读写缓存区时，只需要修改pos,last指针即可。

```c
/*
src/core/ngx_buf.h 
*/
typedef void *            ngx_buf_tag_t;

typedef struct ngx_buf_s  ngx_buf_t;

struct ngx_buf_s {
    u_char          *pos;   /* 缓冲区数据在内存的起始位置 */
    u_char          *last;  /* 缓冲区数据在内存的结束位置 */
    /* 这两个参数是处理文件时使用，类似于缓冲区的pos, last */
    off_t            file_pos;
    off_t            file_last;

    /* 由于实际数据可能被包含在多个缓冲区中，则缓冲区的start和end指向
     * 这块内存的开始地址和结束地址，
     * 而pos和last是指向本缓冲区实际包含的数据的开始和结尾
     */
    u_char          *start;         /* start of buffer */
    u_char          *end;           /* end of buffer */
    ngx_buf_tag_t    tag;
    ngx_file_t      *file;          /* 指向buffer对应的文件对象 */
    /* 当前缓冲区的一个影子缓冲区，即当一个缓冲区复制另一个缓冲区的数据，
     * 就会发生相互指向对方的shadow指针
     */
    ngx_buf_t       *shadow;

    /* 为1时，表示该buf所包含的内容在用户创建的内存块中
     * 可以被filter处理变更
     */
    /* the buf's content could be changed */
    unsigned         temporary:1;

    /* 为1时，表示该buf所包含的内容在内存中，不能被filter处理变更 */
    /*
     * the buf's content is in a memory cache or in a read only memory
     * and must not be changed
     */
    unsigned         memory:1;

    /* 为1时，表示该buf所包含的内容在内存中，
     * 可通过mmap把文件映射到内存中，不能被filter处理变更 */
    /* the buf's content is mmap()ed and must not be changed */
    unsigned         mmap:1;

    /* 可回收，即这些buf可被释放 */
    unsigned         recycled:1;
    unsigned         in_file:1; /* 表示buf所包含的内容在文件中 */
    unsigned         flush:1;   /* 刷新缓冲区 */
    unsigned         sync:1;    /* 同步方式 */
    unsigned         last_buf:1;/* 当前待处理的是最后一块缓冲区 */
    unsigned         last_in_chain:1;/* 在当前的chain里面，该buf是最后一个，但不一定是last_buf */

    unsigned         last_shadow:1;
    unsigned         temp_file:1;

    /* STUB */ int   num;
};
```
## ngx_bufs_t
用来表示缓存区的大小和数量。
比如，在配置文件中，我们配置
>gzip_buffers 8 48k
就使用一个ngx_bufs_t 类型变量来报存这个配置
定义如下：
```c
/*
src/core/ngx_buf.h 
*/
typedef struct {
    ngx_int_t    num;
    size_t       size;    //单位：字节
} ngx_bufs_t;
```
## ngx_chain_t
此结构用于管理ngx_buf_t链，使其形成一个单项链表。
定义如下：
```c
/*
src/core/ngx_buf.h 
*/
struct ngx_chain_s {
    ngx_buf_t    *buf;
    ngx_chain_t  *next;
};
```
# ngx高级数据结构
## ngx_hash_t
## ngx_radix_tree_t
## ngx_rbtree_t
## ngx_pool_t