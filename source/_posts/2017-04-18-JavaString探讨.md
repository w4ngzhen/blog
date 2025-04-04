---
layout: post
title: Java String的探讨
date: 2017-04-18
tags: 
- Java
- String
categories: 
- 技术
---

关于String相关内容的学习，历来都是Java学习必不可少的一个经历。

以前一直想要好好总结一下String的相关的知识点，苦于没有时间，终于在今天有一个闲暇的时间来好好总结一下，也希望这文章能够加深我对于String相关内容的理解~（ps:在我看来，学习某些知识点的时候把学到的想到的都记录下一方面能够加深自己学习印象，二者能够锻炼锻炼我的文笔~）

<!-- more -->

首先JVM中存在着一个字符串池String pool，其中保存着很多String对象，这些String对象可以被共享使用，因此这个pool的存在在很多方面提高Java一些String操作的效率。 而这一些都是基于String类是final的，它的值一经创建就不可改变。String pool由String类维护，所以我们可以调用intern()方法来访问字符串池。

接下里我们来看第一个例子：
```java
String s1 = "abc";     
String s2 = "abc";       
System.out.println("s1 == s2 : "+(s1==s2)); //true     
System.out.println("s1.equals(s2) : " + (s1.equals(s2))); //true 
```
在这个例子中，虚拟机进行了哪些操作呢？首先，因为在此s1我是直接使用了”abc”，那么系统会在String pool中创建了一个String对象,他的值就是”abc”，s2进行创建的时候，系统先搜索String pool中是否已经存在了”abc”，由于String pool中已经存在了一个”abc”的String对象，很好，系统直接将s2指向了String pool中的那个”abc”地址。所以下面两个输出都是true。
上面都是使用””来创建String对象的，第二个例子我们使用new来创建：
```java
String s3 = new String("abc");   
String s4 = new String("abc");   
System.out.println("s3 == s4 : "+(s3==s4)); //false
System.out.println("s3.equals(s4) : "+(s3.equals(s4))); //true 
System.out.println("s1 == s3 : "+(s1==s3)); //false   
System.out.println("s1.equals(s3) : "+(s1.equals(s3))); //true  
```
在这里，应当要提一点。对于new String来说，该构造函数的对象接收一个String字符串对象。其实这里进行了两件事情：

步骤一：系统首先会创建一个String对象”abc”，通过String temp = “abc”来创建的（temp只是我自己取得名字，至于到底叫什么，我们是不用关心的），这个创建过程和第一个例子是一样的道理，他会在String pool去寻找'”abc”，然后根据有没有来判断是额外的创建还是直接用String pool中的那个”abc”。

步骤二：new String接受了刚刚的temp参数以后，在堆上去创建一个对象，把刚刚的temp的值”abc”赋给这个在堆中的String对象的值。

步骤三：把s3指向刚刚在堆中的这个String对象。

通过以上的几步我们可以知道，使用new String完全没有直接使用=””来的快。

s4的创建是同样的道理，所以s3、s4最终都各自指向了内容相同，地址不同的，在堆中的String对象。所以又由于==在对于对象的比较的时候是直接比较的引用的地址。所以第一、三个输出是false。但是equals对于String比较的是内容的是否相等，所以第二、四个输出true。

接下来是第三种例子，String中 + 的重载：

由于我们知道，常量的值在编译的时候就已经进行了优化了。所以这一行代码：

 String str1 = "ab" + "cd";
在编译的时候就等价于优化过的

String str1 = "abcd";
所以执行下面这几行代码的时候，根据常量编译优化，加上我对于第一个例子的讲解，我们可以知道输出是true：
```java
String str1 = "ab" + "cd";  //1个对象  
String str2 = "abcd";   
System.out.println("str1 = str2 : "+ (str1 == str2));  //输出为true
```
接下来就是第四种情形了，这种情况依然是String + 的重载，但是比起第三种是有本质的不同的。首先看代码：
```java
String str2 = "ab";
String str3 = "cd";                                       
String str4 = str2 + str3;                                        
String str5 = "abcd";    
System.out.println("str4 = str5 : " + (str4 == str5));
```
解读这一段代码，str2、str3就是情况一，我们不再讨论。此刻str4比起第三种的 + 两端都是String常量而言，这里是对于编译器来说是两个变量。这一个部分，系统会干什么呢？首先，系统会new 一个StringBuilder对象，然后调用该对象的append函数，把str2、str3按顺序连接起来。连接好以后，调用StringBuilder的toString方法，该方法内部new 一个String对象，并把它return。得到str2 + str3的结果，接着str4指向堆中的这个String对象。

注意：对于StringBuilder的创建还有其他的情况。比如假设此处的String str4 = “ab” + “c” + str2 那么，编译器首先会进行一个优化，也就是第三种情况所说的他会优化为String str4 = “abc” + str2 然后，new StringBuilder(“abc”)，然后append(str2)。也就是说，从=号右边开始前部分，他首先会在编译过程判断能否进行优化，可以优化，由优化为最大的常量子串，然后new一个以该最大子串的为初始值的StringBuilder对象，然后再append。

所以，在以上的这四条String创建语句中，创建了str2、str3、str5三个在String pool中的字符串常量，以及在系统内部进行append的StringBuilder，以及toString函数return的在堆中创建的String对象，一共五个对象。

那么最终输出结果当然是false，前者是指向堆中的String对象的地址，而后者是S指向tring pool中String对象的地址。

但是，还有一种变量情况是不一样的：
```java
final String str1 = "b";  
String str2 = "a" + str1;  
String str3 = "ab";  
System.out.println("str2 = str3 : "+ (str2 == str3)); //true
```
这里按照情况四中注意部分来说，输出本来应该是false，但是为什么是true呢？注意看str1前面的修饰符，这里使用了final修饰符，final修饰的变量表示是一个常量，在创建额时候就需要初始化，并且不可修改。在final关键字的修饰下，编译器会在编译过程中就进行优化，其实这里的道理就和例子三是一样的了。这也是为什么这里输出的是true。
