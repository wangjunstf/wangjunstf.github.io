---
title: C语言数组作为函数参数
date: 2021-08-13 01:07:46
updated:
categories: [C]
description: "指针传递：传递的指针只包含数组的起始地址，要使用数组还应传出数组长度信息; 数组名传递：数组名称依旧是一个指针，和方法1本质是一样的，只是书写形式不一样。"
tags: [c,数组]
---
# C语言数组作为函数参数

# 指针传递

**传递的指针只包含数组的起始地址，要使用数组还应传出数组长度信息**

```c
#include <stdio.h>

int arraySum(int *p,int length);

int main(){
    int num[5] = {1,2,3,4,5};
    printf("%d\n",arraySum(num,5));
}

int arraySum(int *p,int length){
    int sum = 0;
    for(int i=0;i<length;i++){
        sum+=p[i];
        printf("p[%d]=%d\n",i,p[i]);
    }
    return sum/length;
}
```

# 数组名传递

**测试数组名称依旧是一个指针，和方法1本质是一样的，只是书写形式不一样。**

```c
#include <stdio.h>

int arrayPrint(int arr[],int len);
int main(){
    int num[5] = {1,2,3,4,5};
    arrayPrint(num,5);
}

int arrayPrint(int arr[],int len){
    for(int i=0; i< len; i++){
        printf("%d ",arr[i]);
    }
    return 0;
}
```

# 二维数组传参

**二维数组的本质是一维数组的数组，即多个一维数组组成的数组，在函数定义中，形参数组必须给出低维数组元素的个数**

```c
#include <stdio.h>

int arrayPrint(int arr[][3],int row,int column);
int main(){
    int array[3][3] = {{1,2,3},{4,5,6},{7,8,9}};
    arrayPrint(array,3,3);
}


int arrayPrint(int arr[][3],int row,int column){
    for(int i=0;i<row;i++){
        for(int j=0;j<column;j++){
            printf("%d ",arr[i][j]);
        }
        printf("\n");
    }
    return 0;
}
```

# 多维数组传参

**多维数组的本质是多个次低维数组组成的数组。类比方法3，多维数组传参要给出次低维数组的所有信息。**

```c
#include <stdio.h>

int arrayprint3(int arr[][3][3],int count,int row,int column);

int main(){
    int arrays[3][3][3] = {{{1,2,3},{4,5,6},{7,8,9}},{{10,11,12},{13,14,15},{16,17,18}},{{19,20,21},{22,23,24},{25,26,27}}};
    arrayprint3(arrays,3,3,3);
}

int arrayprint3(int arr[][3][3],int count,int row,int column){
    for(int i = 0;i<count;i++){
        for(int j=0; j<row; j++){
            for(int k=0; k<column; k++){
                printf("%d ",arr[i][j][k]);
            }
            printf("\n");
        }
        printf("\n");
    }
    return 0;
}
```
