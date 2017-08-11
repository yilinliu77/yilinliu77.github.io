---
title: JavaScript常见集合操作
date: 2017-08-09 21:37:20
tags:
- JavaScript
- collection
- 集合
- 遍历
categories: JavaScript
---

## 集合的遍历

### for循环(效率最高)

- 优点：JavaScript最普遍的for循环，执行效率最高
- 缺点：无法遍历对象

```
for(let i=0;i<array.length,i++){
    //operation
}
```

### for...in循环(效率较低)

- 优点：唯一一个能够获取对象的属性名的遍历方式
- 缺点：会将对象通过继承得到的属性一齐遍历，造成非预料的结果且效率较低

```
//会访问非继承的属性
for(attr in object){//attr作为属性名
    //object[attr]访问值
}
```

```
//避免访问继承的属性
for(attr in object){//attr作为属性名
    if(object.hasOwnProperty(attr)){
        //object[attr]访问值        
    }
}
```

### for...of循环(效率较高)

- 优点：能够快速访问非继承属性值
- 缺点：需要ES6支持

```
for(item of object){
    //item访问值
}
```

### forEach方法(数组内置高阶方法，含义清晰)

- 优点：函数式编程，简洁，快速领会代码含义
- 缺点：无法对对象使用

```
array.forEach(function(item,index,array)){
    //item为值
    //index为索引
    //array为被访问数组
};
```

### Tips:
1. 在对对象进行遍历时，如不需要访问属性名选择`for...of`循环，如需访问属性名选择`for...in`循环
2. 在对数组进行访问时，使用`forEach`得到较好的可读性，传统的`for`循环能够带来很高的性能及拓展性

## 集合的操作

在小波老师提倡的想机器一样思考中，编程问题的解决被分为了输入，处理和输出
> ![像机器一样思考](http://otbwgn2nv.bkt.clouddn.com/ddbb7a0501334ed5b6fc3cf6adba1c8c.png)
> 引用自[像机器一样思考（一）—— 宏观的基础](https://www.zybuluo.com/jtong/note/403738)

处理，是对输入数据的处理，就可以分为从输入的数据中提炼出一定的有价值的数据，并对他们做出一些操作，得到希望得到的有价值的东西，并将他输出

### Map映射

Map映射是将输入的数据中有价值的东西提取出来，转化为更有利于处理的格式
```
let dataAfterProcess = array.map(function(item,index,array){
    //item为值
    //index为索引
    //array为被访问数组
    return ;//返回dataAfterProcess中希望被添加的元素
});
```
### Reduce计算

Reduce计算以提取好的数据输入，并获得最终的**一个**结果
```
let output = array.reduce(function(accumulator, currentValue, currentIndex, array){
    //accumulator为输出结果
    //currentValue为遍历数组目前的值
    //currentIndex为遍历数组目前的索引
    //array为被访问数组
    return ;//返回希望累加的操作
},0);//0为计算结果的初始值，默认为数组第一个元素
```

## TODO
在完成JS练习中，我时常会遇到以下问题待解决：

1. 在`Map`操作中，经常会遇到需要根据已有的目标数组的情况做出相应的映射操作，但目前尚未发现怎样在Map循环中检查已映射的目标数组
2. 为对象实现接口使对象也具有`MapReduce`操作的能力
