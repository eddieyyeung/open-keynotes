---
description: 浅谈一下 SDS 的数据结构
---

# Simple Dynamic String

之前被朋友问到过 SDS，查了一下才了解到是 redis 的动态字符串的数据结构设计，看了下源码（[https://github.com/antirez/sds](https://github.com/antirez/sds)），觉得有意思，就简单聊一下它的数据结构的设计。

SDS 的基本结构如下所示：

```
+--------+-------------------------------+-----------+
| Header | Binary safe C alike string... | Null term |
+--------+-------------------------------+-----------+
         |
         `-> Pointer returned to the user.
```

**Header** 作为结构体的头部，记录着字符串长度（len）、字符串容量（cap）、数据类型（flag）

**Binary-safe string** 指的是可以包含任意类型的数据的字符串，包括 `\0` 在内，其中会返回指向字符串的指针给用户使用。在 C 语言中，`string` 是一个指向字节数组（比如 `[]*char`）的指针，并且这个字节数组是以一个 Null byte 作为结尾，比如 `\0`。因此，C 中的 string 不是一个 binary-safe string，如果要在 C 语言中构造一个 binary-safe string，通常的结构如下：

```c
struct bytestring {
  size_t len; unsigned char * bytes; 
};
```

**Null term** 就是上文所说的，C 语言中 `string` 隐式代表字符串结束的结束符 `\0`。因此，SDS 的设计兼容了 C 语言的一些常规函数，比如 `printf`。

头文件 [sds.h](https://github.com/antirez/sds/blob/master/sds.h#L45) 是这么声明 SDS 的数据结构的：

```c
/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};

#define SDS_TYPE_5  0
#define SDS_TYPE_8  1
#define SDS_TYPE_16 2
#define SDS_TYPE_32 3
#define SDS_TYPE_64 4
#define SDS_TYPE_MASK 7
#define SDS_TYPE_BITS 3
#define SDS_HDR_VAR(T,s) struct sdshdr##T *sh = (void*)((s)-(sizeof(struct sdshdr##T)));
#define SDS_HDR(T,s) ((struct sdshdr##T *)((s)-(sizeof(struct sdshdr##T))))
#define SDS_TYPE_5_LEN(f) ((f)>>SDS_TYPE_BITS)
```

针对不同长度的字符串用不同的类型来进行存储，分别是：SDS_TYPE\_5、SDS_TYPE\_8、SDS_TYPE\_16、SDS_TYPE\_32、SDS_TYPE\_64。

SDS_TYPE\_5 的数据结构由 flags（低 3 位表示类型，高 5 位表示长度）、buf\[]。其他数据结构由 len（长度）、alloc（容量，即 cap）、flags（低 3 位表示数据类型，高 5 位无用）和 buf\[] 构成。如下图所示。

![](<../.gitbook/assets/image (5).png>)

其中 `s` 表示返回给用户指向存储字符串的指针。由于处理器框架的原因，会导致在创建一个结构体的时候发生数据结构对齐（[data structure alignment](https://en.wikipedia.org/wiki/Data_structure_alignment)），因此源码中使用了 `__attribute__ ((__packed__))` ，这样结构体会以紧凑的字节排列。最终，`s[-1]` 会指向 flags，然后通过与掩码 SDS_TYPE_MASK 进行 `&` 位运算后，即可得出 SDS 的具体类型。

## 参考

* [sds source code](https://github.com/antirez/sds)
* [data structure alignment](https://en.wikipedia.org/wiki/Data_structure_alignment)
* [redis data types - binary safe string](https://redis.io/topics/data-types)

