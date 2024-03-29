---
title: JS 函数定义和调用
date: 2021-08-25 16:31:35
updated:
categories: [JS]
tags: [JavaScript,函数]
---
## 函数定义
```js
function abs(x) {
    if (x >= 0) {
        return x;
    } else {
        return -x;
    }
}
```
<!-- more -->
**function**表示函数定义，本质上是一个函数对象

**abs**表示函数名字

**x**为函数形参，其作用域仅为函数体内

{...}为函数体

**return** 函数返回，表示函数执行完毕

> **return**显式返回**值**
>
> **省略return**时返回**undefined**



## 函数调用

```js
abs(-10);  //返回10
abs(9);    //返回9
abs(4,'hel','wor')  //返回4
abs(-3,'sdf','12')  //返回3
abs();     //返回NaN
```

函数实参(调用时传入的变量)个数，可以**小于等于大于**函数形参(函数定义的参数)个数：

**大于等于**时：用函数实参依次初始化函数形参，多余的不作处理。`abs(4,'hel','wor') `，用4初始化abs，其余的忽略。

**小于**时：未初始化的函数形参，则该参数为undefined。`abs()`，未初始化abs，其值为undefined，此时返回的结果为**NaN**，表示不是一个数字。



## 传递多个参数

可以给函数传递多个参数，事先不确定，可能是1个，2个或多个。

### 获取参数个数

利用arguments可以获取实际调用时传递的参数个数：`arguments.length`

利用下标值获取参数的实际值：`arguments[i]`



### 传递任意个参数

使用rest参数可以接收任意个形参，可以像使用数组一样使用它。

**任意参数个数**

```js
<script>
  			//该函数求任意个数字的和
        function getSum(...rest) {
            var sum = 0;
            for(var i=0; i<arguments.length; i++){
                sum+=arguments[i];
            }
            return sum;
        }
        console.log(getSum(1));
        console.log(getSum(1,2));
        console.log(getSum(1,2,3));
</script>
```



固定参数个数+rest的形式，但rest必须放在最后面。

**固定参数个数+任意参数个数**

```js
<script>
        //求a和b的和，若传递外数字，则一并加上
  			//若传递的参数小于2个，则返回NaN
        function getTwoSum(a,b,...rest) {
            var sum = a+b;
            for(var i=2; i<arguments.length; i++){
                sum+=arguments[i];
            }
            return sum;
        }
        console.log(getTwoSum(1));
        console.log(getTwoSum(1,2));
        console.log(getTwoSum(1,2,3));
</script>
```



## 抛出异常

有时函数需要传入一个整数，但调用者却传入一个字符串怎么办？

这时可以抛出一个异常，例如：

```js
function abs(x){
                   if(typeof x !== 'number'){
                       throw 'Not a number';
                   }

                   if(x>=0){
                       return x;
                   }else{
                       return -x;
                   }
               }
```