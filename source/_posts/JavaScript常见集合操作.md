---
title: JavaScript常见集合操作
date: 2017-08-09 21:37:20
tags:
---

# JavaScript常见集合操作
***
## 集合的遍历

### for循环(效率最高)

- 优点：JavaScript最普遍的for循环，执行效率最高
- 缺点：无法遍历对象

```
for(let i=0;i<array.length,i++){
    //operation
}
```

### for...in循环(效率较高)

- 优点：唯一一个能够获取对象的属性名的遍历方式
- 缺点：会将对象通过继承得到的属性一齐遍历，造成非预料的结果

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
- 缺点：无法访问对象属性名

```
array.forEach(function(item,index,array)){
    //item为值
    //index为索引
    //array为被访问数组
};
```
