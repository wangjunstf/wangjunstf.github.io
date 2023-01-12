---
title: C/C++ 实现一个堆内存分配器(malloc/free)
date: 2021-11-09 18:27:32
updated:
categories: [深入理解计算机系统]
tags: [堆内存,内存分配器,malloc,free]
---
C 语言使用 malloc 分配内存，使用 free 释放内存。那么它们是怎么实现的呢？

堆内存位于数据段(data) 和内存映射区之间，它有一个堆顶指针 brk，malloc 将堆内存分为空闲块和已分配块，使用链表来管理空闲块和已分配块。当堆内存用完时，使用系统调用 sbrk 增大 brk 来增大堆内存的大小。当要求分配的内存大小大于空闲块时，就将空闲块分成两份，一份分配给用户，剩下的内存作为一个空闲块。
<!-- more -->
随着系统的运行，将会产生大量小的空闲块，这些空闲块组合起来可以满足用户要求，但没有一块空闲块可以满足用户要求，这些大量的小的空闲块称为外部碎片。与外部碎片相对的就是内部碎片，内部碎片往往是由于内存对齐的需要，而分配大于要求分配的大小，这些多余的部分称为内部碎片。

下面将使用 C 语言实现一个类似于 malloc 的堆内存分配器，以及类似于 free 的内存释放功能。基于隐式空闲链表，首次适配搜索。[关于隐式空闲链表的详细介绍](https://wangjunstf.github.io/2021/11/09/linux-xu-ni-nei-cun-xi-tong-nei-cun-ying-she-fork-execve-malloc-free-dong-tai-nei-cun-fen-pei-yu-shi-fang/)。

头文件 mm.h

```c
#include <memory.h>
#include <stdlib.h>
#include <errno.h>
#include <stdio.h>

static char *mem_heap;     // 指向堆的第一个字节
static char *mem_brk;      // 指向堆最后一个字节的后面一个字节
static char *mem_max_addr; // 指向堆的最大合法地址的后一个地址
static char *heap_listp;   // 总是指向序言块

#define WSIZE 4
#define DSIZE 8
#define CHUNKSIZE (1<<12)    // 4KB 

#define MAX_HEAP CHUNKSIZE

#define MAX(x,y) ((x) > (y) ? (x) : (y))

// 将大小和状态位打包进一个字(4字节)里
#define PACK(size,alloc) ((size)|(alloc))

// 在一个地址里读取或写入一个字的数据
#define GET(p) (*(unsigned int *)(p))
#define PUT(p, val) (*(unsigned int *)(p)=(val))

// 从一个地址获得块的大小和状态
#define GET_SIZE(p) (GET(p) & ~0x7)
#define GET_ALLOC(p) (GET(p) & 0x1)

// 给定一个块的地址，计算出它的头部和脚部的地址
#define HDRP(bp) ((char*)(bp) - WSIZE)
#define FTRP(bp) ((char*)(bp) + GET_SIZE(HDRP(bp)) - DSIZE)

// 给定一个块的地址，计算出它的下一块和上一块的地址
#define NEXT_BLKP(bp) ((char*)(bp) + GET_SIZE((char*)(bp) - WSIZE))
#define PREV_BLKP(bp) ((char*)(bp) - GET_SIZE((char*)(bp) - DSIZE))

void mem_init();          		 // 初始化堆
void *mm_malloc(size_t size);  // 分配内存
void mm_free(void *bp);				 // 回收内存

int mm_init();     // 初始化空闲链表
static void* extend_heap(size_t words);     // 扩展堆的大小
static void *coalesce(void *bp);						// 合并空闲块
static void* find_fit(size_t asize);				// 首次适配搜索
static void place(void* bp, size_t asize);	// 分割空闲块
```

mm.c

```c
#include "mm.h"

void mem_init(){
    // 为堆分配内存
    mem_heap = (char*)malloc(MAX_HEAP);

    // 初始化堆顶指针
    mem_brk = (char*)mem_heap;

    // 初始化最大合法地址
    mem_max_addr = (char*)(mem_heap+MAX_HEAP);
}

// 扩展堆顶指针
void *mem_sbrk(int incr){
    char* old_brk = mem_brk;
    if( (incr<0) || (mem_brk+incr>mem_max_addr)){
        errno = ENOMEM;
        fprintf(stderr,"ERROR: mem_sbrk failed. Ran out of memory...\n");
        return (void*)-1;
    }

    mem_brk+=incr;
    return (void*)old_brk;
}


int mm_init(){
    // 将堆扩大 16 字节，存放起始块(4字节) 序言块(8字节) 结尾块(4字节) 
    if((heap_listp=mem_sbrk(4*WSIZE)) == (void*)-1)
        return -1;
    
    PUT(heap_listp,0);
    PUT(heap_listp + (1*WSIZE), PACK(DSIZE,1));
    PUT(heap_listp + (2 * WSIZE), PACK(DSIZE, 1));  // 序言块：(8字节：一个头部，一个尾部)
    PUT(heap_listp + (3*WSIZE), PACK(0,1));         // 结尾块
    heap_listp+=(2*WSIZE);                          // heap_listp 总是指向序言块
    
    if(extend_heap(CHUNKSIZE/WSIZE)==NULL)          // 扩展堆 1KB 大小
        return -1;
    return 0;
}

static void *extend_heap(size_t words){
    char* bp;
    size_t size;

    // 为了保持对齐，将请求大小向上舍入为最接近2字(8字节)的倍数
    size = (words%2) ? (words+1) :words;
    if((long)(bp = mem_sbrk(size))==-1)
        return NULL;

    // 初始化空闲块的头部和脚部
    PUT(HDRP(bp),PACK(size,0));
    PUT(FTRP(bp), PACK(size,0));
    PUT(HDRP(NEXT_BLKP(bp)), PACK(0,1));  // 每次扩展堆都要重新设置结尾块

}

void mm_free(void *bp){
    size_t size = GET_SIZE(HDRP(bp));
    PUT(HDRP(bp), PACK(size,0));
    PUT(FTRP(bp), PACK(size,0));
    coalesce(bp);
}

static void *coalesce(void *bp){
  	
    size_t prev_alloc = GET_ALLOC(FTRP(PREV_BLKP(bp)));
    size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(bp)));
    size_t size = GET_SIZE(HDRP(bp));

    if(prev_alloc && next_alloc ){  // 情况1 上下都不空闲
        // printf("case1\n");
        return bp;
    }else if(prev_alloc && !next_alloc){
        // printf("case2\n");
        size += GET_SIZE(HDRP(NEXT_BLKP(bp)));
        PUT(HDRP(bp),PACK(size,0));
        PUT(FTRP(bp),PACK(size,0)); 
    }else if(!prev_alloc && next_alloc){
        // printf("case3\n");
        size += GET_SIZE(HDRP(PREV_BLKP(bp)));
        PUT(FTRP(bp),PACK(size,0));
        PUT(HDRP(PREV_BLKP(bp)),PACK(size,0));
        bp = PREV_BLKP(bp);
    }else{
        // printf("case4\n");
        size += GET_SIZE(HDRP(PREV_BLKP(bp)))+GET_SIZE(FTRP(NEXT_BLKP(bp)));
        PUT(HDRP(PREV_BLKP(bp)),PACK(size,0));
        PUT(FTRP(NEXT_BLKP(bp)),PACK(size,0));
        bp = PREV_BLKP(bp);
    }

    return bp;
}

static void *find_fit(size_t asize){
    void *first = heap_listp;
    while(GET_SIZE(HDRP(first))!=0){
        // printf("find_fit %d\n", GET_SIZE(HDRP(first)));
        if (GET_SIZE(HDRP(first)) >= asize && !GET_ALLOC(HDRP(first)))
        {
            // printf("返回大小：%d\n",GET_SIZE(HDRP(first)));
            return first;
        }
        else
        {
            first = NEXT_BLKP(first);
        }
    }
    
    return NULL;
}

static void place(void *bp, size_t asize){
    size_t rest = GET_SIZE(HDRP(bp))-asize;
    if(rest<2*DSIZE){
        return;
    }else{
        PUT(HDRP(bp),PACK(asize,1));
        PUT(FTRP(bp),PACK(asize,1));
        bp = NEXT_BLKP(bp);
        PUT(HDRP(bp),PACK(rest,0));
        PUT(FTRP(bp),PACK(rest,0));
    }
}

void *mm_malloc(size_t size){
    size_t asize;
    size_t extendsize;
    char* bp;

    if(size == 0){
        return NULL;
    }

    if(size <= DSIZE){
        asize = 2*DSIZE;
    }else{
        asize = DSIZE*( (size+DSIZE + DSIZE-1)/DSIZE);
    }

    if((bp = find_fit(asize)) != NULL){           // 找到合适的块
        place(bp,asize);   
        return bp;
    }

    extendsize = MAX(asize,CHUNKSIZE);
    if((bp=extend_heap(extendsize/WSIZE))==NULL) // 没找到合适的块
        return NULL;
    
    place(bp,asize);
    return bp;
}
```

测试

main.c

```c
#include <stdio.h>
#include <string.h>
#include "mm.h"

int main(){
    mem_init();
    mm_init();

    int *p = mm_malloc(sizeof(int));
    char *s = mm_malloc(50);
    double *num = mm_malloc(sizeof(double));

    *p = 12;
    strncpy(s, "Hello world", 50);
    *num = 0.00012;

    printf("%d ---> %p\n",*p,p);
    printf("%lf ---> %p\n", *num, num);
    printf("%s ---> %p\n", s, s);

    mm_free(p);
    p = mm_malloc(sizeof(int));
    *p = 1024;
    printf("%d ---> %p\n", *p, p);

    mm_free(s);
    mm_free(num);
    return 0;
}
```

输出：

```
12 ---> 0x55c888a7b270
0.000120 ---> 0x55c888a7b2c0
Hello world ---> 0x55c888a7b280
1024 ---> 0x55c888a7b270
```

