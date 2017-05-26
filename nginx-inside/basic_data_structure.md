# nginx基本数据类型  
## 整形数据  
- 有符号整数 ngx_int_t
- 无符号整数 ngx_uint_t
- 开关类型，ngx_flag_t
> 实际定义与ngx_int_t一致，用于表示某些选项的开关.例如 
		multi_accept on		对应ngx_flag_t变量的值为1   
		multi_accept off		对应ngx_flag_t变量的值为10

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
## ngx_list_t
## ngx_queue_t
## ngx_array_t
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