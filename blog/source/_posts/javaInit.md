---
title: javaInit
date: 2021-06-27 14:22:37
cover: https://i.loli.net/2021/06/27/K5JUjf3ylB1dILD.jpg
tags:
 - 八股战神系列
---

# Java常见核心命题



## 本章思维导图

{% asset_img image-20210627142649358.png 思维导图 %}

## 基础概念

1. **抽象与接口**

   * 抽象是对类的抽象，为了解决代码复用，表示is-a的关系。是一种自下而上的设计思路，先有子类的代码重复，才有抽象上层父类；
   * 接口是对行为的抽象，侧重于解耦，实现了约定和实现的分离，表示has-a的关系。是一种自上而下的设计思路，先设计好接口，再考虑具体实现。

2. **```final```关键字**

   * final修饰变量：不可被修改
   * final修饰参数：在函数里不可被修改
   * final修饰方法：不可被重写（private级别的方法默认都是final）
   * final修饰类：不可被继承
   * **final修饰对象类型变量**，仅引用不可变

3. **```static```关键字**

   * 静态变量：类变量（属于类而非属于实例），所有实例共享，在内存中只有一份，会在类初始化的时候赋系统默认值（final修饰的除外）。**基本类型**会将值存储在方法区，而**引用类型**在方法区存储的是引用/堆地址
   * 静态方法：必须有实现，不可为抽象方法。
   * 静态代码块：仅类初始化运行一次，先父后子。
   * 静态内部类：
     * 在外部类的非静态方法中，使用方式没有差别
     * 在外部类的静态方法以及在其他类中，如果要使用，非静态内部类需要new Outter().new Inner();来实现，而静态内部类只需new Inner()即可。故，内部类都写成静态的
   * 静态导包：静态方法或者变量的使用方式是Class.method/var，采用静态导包之后，可以直接使用method而不必指明Class

4. **泛型**

   * 概念：参数化类型（区别于参数化变量）；仅在编译阶段有效，编译时的类型安全检测机制；运行期泛型消失（类型擦除）
   * 分类：泛型类 & 泛型接口 & 泛型方法
   * 类型通配符：
     * ```<?>```无界通配符
     * ```? extends Type``` 上边界通配符，只读不写，如list只能get而不能add()
     * ```? super Type``` 下边界通配符，只写不读，如list只能返回Object，作为函数返回类型没有意义
     * 通配符限定的范围是在确定"**参数化类型**"的时候

5. **缓存池**

   * 基本类型缓存池，如果对应包装类型的数值范围在缓存池内，直接使用缓存池中的对象

     * Code Demo: 
     
     * ```java
       Integer a = 123; // Integer.valueOf(123)
       Integer b = 123;
       System.out.println(a == b); // true
       Integer a = new Integer(123);
       Integer b = new Integer(123);
       System.out.println(a == b); // false
       Integer a = 129;
       Integer b = 129;
       System.out.println(a == b); // false
       ```
       Integer/short的范围为-128～127
     
   * String类型缓存池，保存所以String字面量，同时还可以inter()进行添加

     * inter(): 添加进String pool中返回该字符串的引用

## HashMap

较为详细的核心原理请参考[徒手撸轮子系列之HashMap——我们撸一个HashMap怎么样](https://yzhao.top/2021/04/27/myHashMap/)

