---
layout: post
title:  "Java的lambda表达式----浅谈"
date:   2020-4-10
categories: java
tags: java
author: Clear Li
---

lambda是jdk1.8才有的。第一次见这种表达式的时候，我也是很难接受。一直使用的很笨的方式创建，再加上idea也能自动补全也就一直没有关注这个写法。。。下面简单记录一下。等之后在完善吧。









[TOC]



## 搞几个小例子

首先先看之前创建线程的代码

```java
new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("测试");
            }
        }).start();

```

这是创建一个线程的一种方式----实现**Runnable接口**，这个不是重点。关键就是代码看起来比较冗余。实际有用的代码其实只有一行。废话不说了，直接上**lambda的写法**。

```java
 new Thread(()->{
            System.out.println(123);
        }).start();a
```

总体来说lambda有两个

- （）括号内容 ：里面可以传入参数，但也不是瞎传。下面详细说
-    {} 括号内容：具体实现的方法

下面再看一个例子

```java
 interface Math{
        int mathTo(int a, int b );
    }

 private int  getOperate(int a,int y,Math math){
        return math.mathTo(a,y);
    }
```



首先定义一个接口（因为lambda表达式一般是实现一个接口）。现在这个getOperate只是告诉a和y来被Math操作一番。那么我们要是使用这个方法的话必须实现这个接口。

```java
getOperate(1, 2, (int x, int y) -> x + y );
       
```

只需上面添加一个表达式即可。

下面再看一个经常使用的类---Comparator

```java
Comparator<User> comparator = new Comparator<User>(){
            @Override
            public int compare(User o1, User o2) {
                return o1.getId()-o2.getId();
            }
        };
Comparator<User> comparator2 =
                (o1, o2) -> o1.getId()-o2.getId();
 Comparator<User> comparator3 = Comparator.comparing(User::getId);
        User[] users = new User[12];
        //users1.sort(comparable);
        Arrays.sort(users, Comparator.comparing(User::getId));
```

一共三种实现方式，前两种跟刚才一模一样，主要就是第三种实现方式，这个双冒号，下面是摘录的

[这里]: https://blog.csdn.net/weixin_42740530/article/details/104655470

认为讲的还不错

`::`关键字提供了四种语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与`lambda`联合使用，`::`关键字可以使语言更简洁，减少冗余代码。

| 语法种类                         | 示例                                 |
| :------------------------------- | ------------------------------------ |
| 引用静态方法                     | ContainingClass::staticMethodName    |
| 引用特定对象的实例方法           | containingObject::instanceMethodName |
| 引用特定类型的任意对象的实例方法 | ContainingType::methodName           |
| 引用构造函数                     | ClassName::new                       |



- 



## 引用静态方法

```java
public class Person {

    public enum Sex {
        MALE, FEMALE
    }

    String name;
    LocalDate birthday;
    Sex gender;
    String emailAddress;

    public int getAge() {
        // ...
    }
    
    public Calendar getBirthday() {
        return birthday;
    }    

    public static int compareByAge(Person a, Person b) {
        return a.birthday.compareTo(b.birthday);
    }}
```

假设您的社交网络应用程序的成员包含在一个数组中，并且您想按年龄对数组进行排序。您可以使用以下代码（在示例中找到本节中描述的代码摘录 [MethodReferencesTest](https://docs.oracle.com/javase/tutorial/displayCode.html?code=https://docs.oracle.com/javase/tutorial/java/javaOO/examples/MethodReferencesTest.java)）：

```java
Person[] rosterAsArray = roster.toArray(new Person[roster.size()]);

class PersonAgeComparator implements Comparator<Person> {
    public int compare(Person a, Person b) {
        return a.getBirthday().compareTo(b.getBirthday());
    }
}
        
Arrays.sort(rosterAsArray, new PersonAgeComparator());
```

调用的方法签名如下:

```java
static <T> void sort(T[] a, Comparator<? super T> c)
```

请注意，该接口`Comparator`是功能接口。因此，您可以使用`lambda`表达式，而不是定义并创建一个新类的实例，该实例实现`Comparator`：

```java
Arrays.sort(rosterAsArray,
    (Person a, Person b) -> {
        return a.getBirthday().compareTo(b.getBirthday());
    }
);
```

但是，这种用于比较两个`Person`实例的出生日期的方法已经存在`Person.compareByAge`。您可以改为在`lambda`表达式的主体中调用此方法：

```java
Arrays.sort(rosterAsArray,
    (a, b) -> Person.compareByAge(a, b)
);
123
```

由于此`lambda`表达式会调用现有方法，因此您可以使用方法引用代替`lambda`表达式：

```java
Arrays.sort(rosterAsArray, Person::compareByAge);
1
```

方法引用`Person::compareByAge`在语义上与`lambda`表达式相同`(a, b) -> Person.compareByAge(a, b)`。每个都有以下特征：

- 它的形参列表是从复制`Comparator.compare`，这是`(Person, Person)`。
- 它的主体调用该方法`Person.compareByAge`。

## 引用特定对象的实例方法

以下是对特定对象的实例方法的引用示例：

```java
class ComparisonProvider {
    public int compareByName(Person a, Person b) {
        return a.getName().compareTo(b.getName());
    }
        
    public int compareByAge(Person a, Person b) {
        return a.getBirthday().compareTo(b.getBirthday());
    }
}
ComparisonProvider myComparisonProvider = new ComparisonProvider();
Arrays.sort(rosterAsArray, myComparisonProvider::compareByName);
1234567891011
```

方法引用`myComparisonProvider::compareByName`调用`compareByName`作为对象一部分的方法`myComparisonProvider`。`JRE`推断方法类型参数，在这种情况下为`(Person`, `Person)`。

## 引用特定类型的任意对象的实例方法

以下是对特定类型的任意对象的实例方法的引用示例：

```java
String[] stringArray = { "Barbara", "James", "Mary", "John",
    "Patricia", "Robert", "Michael", "Linda" };
Arrays.sort(stringArray, String::compareToIgnoreCase);
123
```

方法参考的等效`lambda`表达式`String::compareToIgnoreCase`将具有形式参数列表`(String a, String b)`，其中`a`和`b`是用于更好地描述此示例的任意名称。方法引用将调用该方法`a.compareToIgnoreCase(b)`。

## 引用构造函数

您可以使用`name`以与**静态方法**相同的方式引用构造函数`new`。以下方法将元素从一个集合复制到另一个：

```java
public static <T, SOURCE extends Collection<T>, DEST extends Collection<T>>
    DEST transferElements(
        SOURCE sourceCollection,
        Supplier<DEST> collectionFactory) {
        
        DEST result = collectionFactory.get();
        for (T t : sourceCollection) {
            result.add(t);
        }
        return result;
}
1234567891011
```

功能接口`Supplier`包含一个`get`不带任何参数并返回一个对象的方法。因此，您可以`transferElements`使用`lambda`表达式调用该方法，如下所示：

```java
Set<Person> rosterSetLambda =
    transferElements(roster, () -> { return new HashSet<>(); });
12
```

您可以使用构造函数引用代替`lambda`表达式，如下所示：

```java
Set<Person> rosterSet = transferElements(roster, HashSet::new);
1
```

`Java`**编译器**推断您要创建一个`HashSet`包含`type`元素的集合`Person`。或者，您可以指定以下内容：

```java
Set<Person> rosterSet = transferElements(roster, HashSet<Person>::new);
```

个人感受

我感觉lambda还是很有用的比较直观，代码量少。但是对于这个双引号来说，其实有点难懂反而让我感觉起到了相反的效果，可能是我用的比较少吧。。。

