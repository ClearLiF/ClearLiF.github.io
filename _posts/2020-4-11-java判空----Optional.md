---
layout: post
title:  "java判空----Optional"
date:   2020-4-11 11:40:18 +0800
categories: java
tags: java
author: Clear Li
---



### 引入

如果从前端传来的值不判断就执行操作那么可能会出现安全问题，或者有时候值就是一个空值。那么我们会首先判断一下如下所示









```java
if (user != null){
   // do something
   if(user.address!=null){
   		//do something
   }
}
```

从上看来代码还是可以接受。但是假如每一个值都这样判断呢？我一直想的肯定有办法去解决的。springboot的有一个判空注解可以做到这一点。但是今天发现一个别的判空方式。

### Optional类API

这个判空类是jdk1.8新引入的。下面是他的api

![](../img/123.png)

最重要的就是of和ofNullable对象，差别在于of不允许参数是null，而ofNullable则无限制。使用时首先**创建Optional对象**，将传入参数的判断交给**Optional**去执行。相当于Optional帮你做处理。下面是例子⬇⬇⬇⬇⬇

### 应用一

```java
  User user = new User();
  Optional.ofNullable(user).ifPresent((u)->{
           System.out.println("u = " + u);
       });
```

简单解释一下，ofNullable判断传入对象是否为空，ifPresent判断传入参数(user)是否存在，存在则执行后面的代码。

### 应用二

```java
User user = new User();
user = Optional.ofNullable(user).orElseGet(User::new);
```

假如说传入对象为空，那么要怎么做呢？------调用orElseGet函数在创建一个，或者做一些其他操作。

### 应用三

```java
  User user = new User();
user.setAddress("河北省");
 Optional.ofNullable(user).map(User::getAddress).ifPresent(System.out::println);
```

假如说要调用传入对象的内部对象呢？比如上面的例子调用地址，但是地址万一也是空的呢。使用map函数调用，如果内部调用为空则也返回为空值。这样在外面在调用ifPresent去做**非空**操作

### 应用四

```java

Optional.ofNullable(user)
        .filter(user1 -> "河北省".equals(user1.getAddress()))
        .orElseGet(()->{
            User u = new User();
            u.setAddress("河北省");
            return u;

        });
```

这里加了一个过滤器，假如传入地址为期望的地址，则放行。假如说传入的user的地址不是希望的位置，则执行orElseGet里面实现的方法。。。。。这里只是举个小栗子，不是一定要这种格式再写。等我多使用使用再完善这篇文章吧。。