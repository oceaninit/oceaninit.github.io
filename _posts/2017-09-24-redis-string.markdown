---
layout: post
title:  "Redis 字符串"
date:   2017-09-24
categories: redis
---


Redis字符串是二进制安全的。二进制安全是指字符串中不能有'\0'，C语言函数库默认字符串是以’\0’结尾的，sdshdr成员len决定了字符串长度而不是c语音的‘\0’，所以Redis中字符串可以存储二进制数据如图片等。实际sds应该是指向buf的。


```
typedef char *sds;

struct sdshdr {
    int len;
    int free;
    char buf[];
};
```

长度计算：O（1）时间复杂度，C语言中strlen的时间复杂度是O（N）。
```
static inline size_t sdslen(const sds s) {
    struct sdshdr *sh = (void*)(s-(sizeof(struct sdshdr)));
    return sh->len;
}

static inline size_t sdsavail(const sds s) {
    struct sdshdr *sh = (void*)(s-(sizeof(struct sdshdr)));
    return sh->free;
}
```

字符串拼接：动态扩容

1、拼接后的字符串长度小于1M(SDS_MAX_PREALLOC)，长度扩容为2倍，否则；

2、扩容为拼接后字符串的长度 + SDS_MAX_PREALLOC


```
#define SDS_MAX_PREALLOC (1024*1024)

sds sdscat(sds s, const char *t) {
    return sdscatlen(s, t, strlen(t));  /* 注意strlen, \0结尾的字符串 */
}

sds sdscatlen(sds s, const void *t, size_t len) {
    struct sdshdr *sh;
    size_t curlen = sdslen(s);

    s = sdsMakeRoomFor(s,len);
    if (s == NULL) return NULL;
    sh = (void*) (s-(sizeof(struct sdshdr)));
    memcpy(s+curlen, t, len);
    sh->len = curlen+len;
    sh->free = sh->free-len;
    s[curlen+len] = '\0';
    return s;
}

sds sdsMakeRoomFor(sds s, size_t addlen) {
    struct sdshdr *sh, *newsh;
    size_t free = sdsavail(s);
    size_t len, newlen;

    if (free >= addlen) return s;
    len = sdslen(s);
    sh = (void*) (s-(sizeof(struct sdshdr)));
    newlen = (len+addlen);
    if (newlen < SDS_MAX_PREALLOC)
        newlen *= 2;
    else
        newlen += SDS_MAX_PREALLOC;
    newsh = zrealloc(sh, sizeof(struct sdshdr)+newlen+1);
    if (newsh == NULL) return NULL;

    newsh->free = newlen - len;
    return newsh->buf;
}
```
