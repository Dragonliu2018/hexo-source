---
title: 《代码整洁之道 clean code》笔记
tags:
  - 重构
categories:
  - [阅读]
  - 软件开发
toc: true
mathjax: true
top: false
comments: true
copyright: true
date: 2022-04-15 15:43:14
---

> * 《代码整洁之道 Clean Code》Robert C. Martin 著 韩磊 译 人民邮电出版社 2020
> * 《Clean Code(评注版)》Robert C. Martin 著 韩磊 评注 电子工业出版社 2012

# Ch2 Meaningful Names(有意义的命名)

## 2.1 Use Intention-Revealing Names(名副其实)

如果名称需要注释来补充，那就算不上名副其实。

```c
int d;//消逝的时间，以日计
int elapsedTimeInDays;
```

## 2.2 Avoid Disinformation(避免误导)

程序员必须避免留下掩藏代码本意的错误线索。应当避免使用与本意相悖的词。

* 缩写形式常常带来误导。编写三角计算程序，hp是hypotenuse(直角三角形的斜边)缩写，但也是UNIX平台或类UNIX平台的专有名称。

* 一些词对于程序员有特殊含义，如List：

  ```c
  accountList;//一组账号，如果账号的容器并List，就会引起错误的判断。
  // 可以换成
  accountGroup;
  bunchOfAccounts;
  ```

* 不使用差异较小的名称，不易区分

  ```c
  XYZControllerForEfficientHandlingOfStrings;
  XYZControllerForEfficientStorageOfStrings;
  ```

* 不用小写字母`l`和大写字母`O`做为变量名，容易与数字`1`和`0`混淆

## 2.3 Make Meaningful Distinctions(做有意义的区分)

* 以数字系列命名：`a1,a2,a3...aN`
* 废话：Product类、ProductInfo类、ProductData类三者名称不同，但是意思无区别

## 2.4 Use Pronounceable Names(使用可读命名)

```c
Data genymdhms;//生成日期，年、月、日、时、分、秒
Data generationTimestamp;//改进
```

## 2.5 Use Searchable Names(使用可搜索命名)

不使用单字母名称和数字常量，在代码中找数字`7`要比找`MAX_CLASSES_PER_STUDENT`容易多；单字母名称可用在短方法的本地变量(local variables inside short methods)，如循环中的计数因子`i`，名称长短应与其作用域大小相对应。

## 2.6 Avoid Encoding(避免编码)

把类型或作用域加进命名中，徒然增加了读懂命名的负担。带编码的名称也不便发音，容易打错。

* 匈牙利语标记法：

  * 在变量名中用缩写标记变量的类型。
  * 在Windows的C语言API时代，编译器并不做类型检查，程序员需要匈牙利语标记法来帮助自己记住类型。
  * 现代编程语言具有更丰富的类型系统，编译器也记得并强制使用类型。而且，人们趋向于使用更小的类、更短的方法，好让每个变量的定义都在视野范围之内。所以，如今HN和其他类型的编码方式纯属多余。

  ```c
  PhoneNumber phoneString;//类型发生变化时，名称并不发生变化
  ```

* 成员前缀：不必使用`m_`前缀来表明成员变量

* 接口和实现(Interfaces and Implementations)

  ```c++
  // build an Abstract Factory for the creation of shapes
  ShapeFactory;//比较好
  IShapeFactory;//I前缀表示接口，是废话
  ShapeFactoryImp;//实现比接口好一些
  ```

## 2.7 Avoid Mental Mapping(避免惯性思维)

不应当让读者在脑中把你的名称翻译为他们熟知的名称；这种问题经常出现在选择是使用问题领域术语还是解决方案领域术语时。

> 聪明程序员和专业程序员之间的区别在于，专业程序员了解，明确是王道。专业程序员善用其能，编写其他人能理解的代码。

## 2.8 Class Names(类名)

* 类名和对象名应该是名词或名词短语，如Customer、WikiPage、Account 和AddressParser。
* 避免使用Manager、Processor、Data或Info这样的类名。
* 类名不应当是动词。

## 2.9 Method Names(方法名)

* 方法名应当是动词或动词短语，如postPayment、deletePage 或save。

* 属性访问器、修改器和断言(accessors, mutators, predicates)应该根据其值命名，并依Javabean 标准1加上get、set和is前缀。

  ```c++
  string name=employee.getName();
  customer.setName("mike");
  if(paycheck.isPosted())...
  ```

* 重载构造器时，使用描述了参数的静态工厂方法名(static factory methods)。例如：

  ```c
  Complex fulcrumPoint=Complex.FromRealNumber(23.0);
  // 通常好于
  Complex fulcrumPoint=new Complex(23.0);
  ```

  可以考虑将相应的构造器设置为private，强制使用这种命名手段。

## 2.10 Don't Be Cute(别抖机灵)

别用俗语、俚语。

## 2.11 Pick One Word per Concept(每个概念对应一个词)

给每个抽象概念选一个词，并且一以贯之。例如：

* fetch、retrieve和get虽有区别，但在前后一致、团队一致的情况下，可以任取其一，一以贯之。
* 还有controller、manager、driver

## 2.12 Don't Pun(别用双关语)

避免将同一单词用于不同目的。“一词一义”原则

## 2.13 Use Solution Domain Names(使用解决方案领域名称)

只有程序员才会读你的代码，所以尽量用那些计算机科学（Computer Science，CS）术语、算法名、模式名、数学术语。

## 2.14 Use Problem Domain Names(使用问题领域名称)

如果不能用程序员熟悉的术语来命名，就采用从所涉问题领域而来的名称。这样负责维护代码的程序员就能去请教领域专家了。

> 优秀的程序员和设计师，其工作之一就是分离解决方案领域和问题领域的概念。

## 2.15 Add Meaningful Context(添加有意义的语境)

用有良好命名的类、函数或名称空间来放置名称给读者提供语境，如果没这么做，给名称添加前缀。比如：firstName、lastName、street、houseNumber、city、state和zipcode的变量

1. 如果他们在一起，很明确是构成了一个地址；
2. 如果只有state，那么很难猜测是地址的一部分；
3. 可以添加前缀addrFirstName、addrLastName、addrState等，以此提供语境；
4. 更好的方案是创建名为Address的类。

## 2.16 Don't Add Gratutous Context(不要添加没用的语境)

只要短名称足够清楚，就要比长名称好，别给名称添加不必要的语境。

比如：设若有一个名为“加油站豪华版”（Gas Station Deluxe）的应用，每个类不要添加GSD前缀。

# Ch3 Functions(函数)

## 3.1 Small(短小)

* if语句、else语句、while 语句等，其中的代码块应该只有一行。该行大抵应该是一个函数调用语句。这样不但能保持函数短小，而且，因为块内调用的函数拥有较具说明性的名称，从而增加了文档上的价值。
* 这也意味着函数不应该大到足以容纳嵌套结构。所以，**函数的缩进层级不该多于一层或两层**。当然，这样的函数易于阅读和理解。

## 3.2 Do One Thing(只做一件事)

如果函数只是做了该函数名下同一抽象层上的步骤，则函数还是只做了一件事。

只做一件事的函数无法被合理地切分为多个区段。

## 3.3 One Level of Abstraction per Function(每个函数一个抽象层级)

要确保函数只做一件事，函数中的语句都要在同一抽象层级上。

自顶向下读代码：向下规则。让代码拥有自顶向下的阅读顺序，让每个函数后面都跟着位于下一抽象层级的函数，这样一来，在查看函数列表时，就能循抽象层级向下阅读了。

## 3.4 Switch Statements(Switch语句)

利用多态能够确保每个switch都埋藏在较低的抽象层级，而且永远不重复。

## 3.5 Use Descriptive Names(使用描述性的名称)

* 沃德原则：“如果每个例程都让你感到深合己意，那就是整洁代码。” 函数越短小、功能越集中，就越便于取个好名字。
* 长而具有描述性的名称，比短而令人费解的名称好，比描述性的长注释好。
* 命名方式要保持一致。使用与模块名一脉相承的短语、名词和动词给函数命名。

## 3.6 Function Arguments(函数参数)

最理想的参数数量是零（零参数函数），其次是一（单参数函数），再次是二（双参数函数），应尽量避免三（三参数函数）。有足够特殊的理由才能用三个以上参数（多参数函数）。

* 标识参数：尽量不向函数传入布尔值；

* 参数对象：如果函数看来需要两个、三个或三个以上参数，就说明其中一些参数应该封装为类了。例如，下面两个声明的差别：

  ```c
  Circle makeCircle(double x, double y, double radius);
  Circle makeCircle(Point center, double radius);
  ```

* 参数列表：有时，我们想要向函数传入数量可变的参数。例如，String.format方法：

  ```java
  String.format("s worked $.2f hours.", name, hours);
  ```

  如果可变参数像上例中那样被同等对待，就和类型为List的单个参数没什么两样。这样一来，String.formate实则是二元函数。下列 String.format的声明也很明显是二元的：

  ```java
  public String format(String format, Object...args);
  ```

## 3.7 Have No Side Effects(无副作用)

函数承诺只做一件事，但还是会做其他被藏起来的事。有时，它会对自己类中的变量做出未能预期的改动。有时，它会把变量搞成向函数传递的参数或是系统全局变量。无论哪种情况，都是具有破坏性的，会导致古怪的时序性耦合及顺序依赖。

## 3.8 Command Query Separation(分隔指令和询问)

函数要么做什么事，要么回答什么事，但二者不可得兼。函数应该修改某对象的状态，或是返回该对象的有关信息。两样都干常会导致混乱。

1. 有两种指令与查询混杂的情况：查询某个状态，再根据状态执行操作；执行某个操作，返回该操作的执行情况。
2. 最常见的就是函数执行某项操作，其布尔类型的返回值代表操作成功与否。

## 3.9 Prefer Exceptions to Returning Error Codes(使用异常代替返回错误码)

## 3.10 Don't Repeat Yourself(别重复自己)

重复可能是软件中一切邪恶的根源。许多原则与实践规则都是为控制与消除重复而创建。例如，

* 考德（Codd）数据库范式都是为消灭数据重复而服务；
* 面向对象编程是如何将代码集中到基类，从而避免了冗余。
* 面向方面编程（Aspect Oriented Programming）、面向组件编程（Component Oriented Programming）多少也都是消除重复的一种策略。

## 3.11 Structured Programming(结构化编程)

* Dijkstra认为，每个函数、函数中的每个代码块都应该有一个入口、一个出口。意味着在每个函数中只该有一个return语句，循环中不能有break 或continue 语句，而且不能有任何 goto语句。
* 但对于小函数，这些规则助益不大。只有在大函数中，这些规则才会有明显的好处。
* 所以，只要函数保持短小，偶尔出现的returm、break 或continue语句没有坏处，甚至还比单入单出原则更具有表达力。另外一方面，goto只在大函数中才有道理，所以应该尽量避免使用。

## 3.12 如何写出这样的函数

并不是一开始就按照规则写函数，需要打磨。

# Ch4 Comments(注释)

1. 注释最多也就是一种必须的恶。若编程语言足够有表达力，或者我们长于用这些语言来表达意图，就不需要注释。
2. 代码在发动，演化；注释不总是随之变动；而代码是唯一真正准确的信息来源。

## 4.1 Comments Do Not Make Up for Bad Code(注释不能弥补糟糕的代码)

## 4.2 Explain Yourself in Code(用代码来阐述)

```java
//方式1
//Check to see if the employee is eligible for full benefits 
if ((employee.flags & HOURLY_FLAG) && (employee.age >65))
//方式2
if (employee.isEligibleForFu11Benefits())
```

## 4.3 Good Comments(好注释)

有些注释是必须的，也是有利的。

1. 法律信息：

   ```java
   //Copyright (C)2003,2004,2005 by Object Mentor, Inc. All rights reserved.
   //Released under the terms of the GNU General Public License version 2 or later.
   ```

2. 提供信息的注释。以下注释解释了某个抽象方法的返回值：

   ```java
   //Returns an instance of the Responder being tested.
   protected abstract Responder responderInstance();
   ```

3. 对意图的解释

4. 阐释：有时，注释把某些晦涩难明的参数或返回值的意义翻译为某种可读形式，也会是有用的。这类注释在低层抽象中较为常见，也应该只出现在低层抽象中。

   ```java
   public void testCompareTo() throws Exception 
   {
       WikiPagePath a=PathParser. parse("PageA"); 
       WikiPagePath ab=PathParser. parse("PageA. PageB");
       WikiPagePath b=PathParser. parse("PageB"); 
       WikiPagePath aa=PathParser. parse("PageA. PageA"); 
       WikiPagePath bb=PathParser. parse("PageB. PageB");
       WikiPagePath ba=PathParser. parse("PageB. PageA"); 
       
       assertTrue(a. compareTo(a)==0);//a==a
       assertTrue(a. compareTo(b)!=0);//a!=b 
       assertTrue(ab. compareTo(ab)==0);//ab==ab 
       assertTrue(a. compareTo(b)==-1);//a<b 
       assertTrue(aa. compareTo(ab)==-1);//aa< ab 
       assertTrue(ba. compareTo(bb)==-1);//ba<bb 
       assertTrue(b. compareTo(a)==1);//b>a 
       assertTrue(ab. compareTo(aa)==1);//ab>aa 
       assertTrue(bb. compareTo (ba)==1);//bb>ba
   }
   ```

5. 警示：有时，用于警告其他程序员会出现某种后果的注释也是有用的。

6. TODO注释：有时用//TODO形式在源代码中放置要做的工作列表。如今，大多数好IDE都提供了特别的手段来定位所有TODO注释，所以TODO注释变得多余。

7. 放大(Amplification)：注释可以用来放大某种看来不合理之物的重要性。

   ```java
   String listItemContent=match.group(3).trim();
   //the trim is real important. It removes the starting//spaces that could cause the itam to be recognized
   //as another list.
   new ListItemwidget(this, listItemContent, this.level+1); 
   return buildList(text.substring(match. end()));
   ```

8. 公共API中的Javadoc

## 4.4 Bad Comments(坏注释)

1. Mumbling：自言自语的注释，往往是作者边写代码边整理思路时留下的。

2. 多余的注释(Redundant Comments)：对足够小的函数不需要再做注释，但是一些关键业务函数加上“他是用来做什么”之类的注释。

3. 误导性注释(Misleading Comments)

4. 循规式注释(Mandated Comments)：每个函数都要有Javadoc 或每个变量都要有注释的规矩是愚蠢可笑的。

5. 日志式注释(Journal Comments)：很久以前，在模块开始处创建并维护这些记录还算有道理。那时，我们还没有源代码控制系统可用。如今，这种冗长的记录只会让模块变得凌乱不堪，应当全部删除。

6. 废话注释(Noise Comments)

7. 可怕的废话(Scary Noise)：Javadoc 也可能是废话。

8. Don't Use a Comment When You can Use a Function or a Variable.

9. 位置标记(Position Markers)

10. 括号后面的注释

    ```java
    while (...) {
        ...
    }//while
    ```

11. 归属与署名：源代码控制系统非常善于记住是谁在何时添加了什么，源代码控制系统是这类信息最好的归属地。

    ```java
    /*Added by Rick*/
    ```

12. 注释掉的代码：20世纪60年代，曾经有那么一段时间，注释掉的代码可能有用，但现在源代码控制系统记住不要的代码。
13. HTML注释
14. 非本地信息：假如你一定要写注释，请确保它描述了离它最近的代码，别在本地注释的上下文环境中给出系统级的信息。
15. 信息过多：别在注释中添加有趣的历史性话题或者无关的细节描述。
16. 不明显的联系：注释及其描述的代码之间的联系应该显而易见。
17. 函数头：短函数不需要太多描述。为只做一件事的短函数选个好名字，通常要比写函数头注释要好。
18. 非公共代码中的Javadoc

# Ch5 Formatting(格式)

## 5.1 Vertical Formatting(垂直格式)

有可能用大多数为200行、最长500行的单个文件构造出色的系统。尽管这并非不可违背的原则，也应该乐于接受。短文件通常比长文件易于理解。

1. Vertical Openness Between Concepts(间隔)：在封包声明、导入声明和每个函数之间，都有空白行隔开。每个空白行都是一条线索，标识出新的独立概念。

2. Vertical Density(靠近)：紧密相关的代码应该互相靠近

3. Vertical Distance(垂直距离)：

   * 变量声明：变量声明应尽可能靠近其使用位置。因为函数很短，本地变量应该在函数的顶部出现。循环中的控制变量应该总是在循环语句中声明。

   * 实体变量应该在类的顶部声明。关于实体变量应该放在哪里，争论不断。在C++中，通常会采用所谓“剪刀原则”（scissors rule），所有实体变量都放在底部。而在Java中，惯例是放在类的顶部。

     > 关于“剪刀原则”较为确切的解释：在类的头文件中，公共部分放在前面，私有部分放到后面，这样就可以用剪刀将文件一分为二，上半部分是该类的可用部分。

   * 相关函数：若某个函数调用了另外一个，就应该把它们放到一起，而且**调用者应该尽可能放在被调用者上面**。

   * 概念相关：概念相关的代码应该放到一起。相关性越强，彼此之间的距离就该越短。这些函数有着极强的概念相关性，因为他们拥有共同的命名模式，执行同一基础任务的不同变种。互相调用是第二位的。即便没有互相调用，也应该放在一起。

## 5.2 Horizontal Formatting(横向格式)

* 尽力保持代码行短小。死守80个字符的上限有点僵化，不反对代码行长度达到100个字符或120个字符。

* 无需拖动滚动条到右边的原则。但近年来显示器越来越宽，而年轻程序员又能将显示字符缩小到如此程度，屏幕上甚至能容纳200个字符的宽度。作者的上限是**120**个字符。

1. Horizontal Openness and Density(区隔与靠近)：使用空格字符将彼此紧密相关的事物连接到一起，也用空格字符把相关性较弱的事物分隔开。请看以下函数：

   ```java
   private void Func(String line){//不在函数名和左圆括号之间加空格,是函数与其参数密切相关；把参数隔开，是参数互相分离的
   	lineCount++;
       int lineSize = line.length(); //在赋值操作符周围加上空格字符，分隔表达式左边、右边
       (-b + c) / (2*a);//空格表示运算顺序
   }
   ```

2. 水平对齐：汇编语言程序使用水平对齐来强调某些程序结构，但是C、C++、Java不需要。

   ```c++
   int    intNumber;
   float  floatNumber;
   double doubleNumber;
   ```

3. 缩进：在短小的if 语句、while循环或小函数中不要违反缩进规则。

   ```c++
   if (...) {return ;}
   ```

4. 空范围：while或for语句的语句体为空，尽量不使用。如果无法避免，就确保空范围体的缩进，用括号包围起来。

   ```c++
   //方式1
   while (...)
   {
       
   }
   //方式2
   while (...)
   ;
   ```

## 5.3 Team Rules(团队规则)

# Ch6 Objects and Data Structures(对象和数据结构)

## 6.1 Data Abstraction(数据抽象)

数据本体是否一定要隐藏起来，也要视使用场合而定；公共变量存在的原因就是其他类总会有存取他的需要。

不要乱加取值器和赋值器而曝露其内部结构。

## 6.2 Data/Objects Anti-Symmetrey(数据、对象的反对称性)

对象把数据隐藏于抽象之后，曝露操作数据的函数。数据结构曝露其数据，没有提供有意义的函数。

## * 6.3 The Law of Demeter(得墨忒耳律)

著名的得墨忒耳律（The Law of Demeter）认为，模块不应了解它所操作对象的内部情形。
更准确地说，得墨忒耳律认为，类C的方法f只应该调用以下对象的方法：

* C
* 由f创建的对象；
* 作为参数传递给f的对象；
* 由C的实体变量持有的对象。

方法不应调用由任何函数返回的对象的方法。

## 6.4 Data Transfer Objects(数据传送对象)

最为精练的数据结构，是一个只有公共变量、没有函数的类。这种数据结构有时被称为数据传送对象，或DTO（Data Transfer Objects）。DTO是非常有用的结构，尤其是在与数据库通信、或解析套接字传递的消息之类场景中。在应用程序代码里一系列将原始数据转换为数据库的翻译过程中，它们往往是排头兵。

# * Ch7 Error Handling(错误处理)

错误处理很重要，但如果它搞乱了代码逻辑，就是错误的做法。

## 7.1 使用异常而非返回码

## 7.2 Write Your Try-Catch-Finally Statement First

## 7.3 Use Unchecked Exceptions(使用不可控异常)

可控异常的代价就是违反开放/闭合原则。如果你在方法中抛出可控异常，而catch 语句在三个层级之上，你就得在catch 语句和抛出异常处之间的每个方法签名中声明该异常。这意味着对软件中较低层级的修改，都将波及较高层级的签名。修改好的模块必须重新构建、发布，即便它们自身所关注的任何东西都没改动过。

## 7.4 Provide Context with Exceptions(给出异常发生的环境说明)

* 抛出的每个异常，都应当提供足够的环境说明，以便判断错误的来源和处所。在Java中，你可以从任何异常里得到堆栈踪迹（stack trace）；然而，堆栈踪迹却无法告诉你该失败操作的初衷。
* 应创建信息充分的错误消息，并和异常一起传递出去。在消息中，包括失败的操作和失败类型。如果你的应用程序有日志系统，传递足够的信息给catch块，并记录下来。

## 7.5 Define Exception Classes in Terms of a Caller's Needs(依照调用者需要定义异常类)

对错误分类有很多方式。可以依其来源分类：是来目组件还是其他地方？或依其类型分类：是设备错误、网络错误还是编程错误？不过，当我们在应用程序中定义异常类时，最重要的考虑应该是他们如何被捕获。

## 7.6 Define the Normal Flow(定义常规流程)

## 7.7 Don't Return Null

## 7.8 Don't Pass Null(不传递null值)

# * Ch8 Boundaries(边界)

## 8.1 Using Third-Party Code(使用第三方代码)

第三方程序包和框架提供者追求普适性，这样就能在多个环境中工作，吸引广泛的用户。而使用者则想要集中满足特定需求的接口，这样会导致系统边界上出现问题。

## 8.2 浏览和学习边界

编写测试来浏览和理解第三方代码。

# Ch9 Unit Tests(单元测试)

## 9.1 TDD三定律

> TDD是**测试驱动开发**（Test-Driven Development）的英文简称，是敏捷开发中的一项核心实践和技术，也是一种设计方法论。TDD的原理是在开发功能代码之前，先编写单元测试用例代码，测试代码确定需要编写什么产品代码。TDD虽是敏捷方法的核心实践，但不只适用于[XP](https://baike.baidu.com/item/XP/776028)（Extreme Programming），同样可以适用于其他开发方法和过程。

三定律：

1. 在编写不能通过的单元测试前，不能编写生成代码；
2. 只可编写刚好无法通过的单元测试，不能编译也算不通过；
3. 只可编写刚好足以通过当前失败测试的生产代码。

## 9.2 Keeping Tests Clean(保持测试整洁)

测试代码与生产代码一样重要。

测试能给软件带来各种好处，flexibilities(可扩展性)、relabilities(可靠性)等。

## 9.3 Clean Tests(整洁的测试)

## 9.4 One Assert per Test(每个测试一个断言)

* 有流派认为，JUnit中每个测试函数都应该有且只有一个断言语句。
* 单个测试中的断言数量应该最小化。

## 9.5 F.I.R.S.T.

* **快速（Fast）**测试应该够快。测试应该能快速运行。测试运行缓慢，你就不会想要频繁地运行它。如果你不频繁运行测试，就不能尽早发现问题，也无法轻易修正，从而也不能轻而易举地清理代码。最终，代码就会腐坏。
* **独立（Independent）**测试应该相互独立。某个测试不应为下一个测试设定条件。你应该可以单独运行每个测试，及以任何顺序运行测试。当测试互相依赖时，头一个没通过就会导致一连串的测试失败，使问题诊断变得困难，隐藏了下级错误。
* **可重复（Repeatable）**测试应当可在任何环境中重复通过。你应该能够在生产环境、质检环境中运行测试，也能够在无网络的列车上用笔记本电脑运行测试。如果测试不能在任意环境中重复，你就总会有个解释其失败的接口。当环境条件不具备时，你也会无法运行测试。
* **自足验证（Self-Validating）**测试应该有布尔值输出。无论是通过或失败，你不应该查看日志文件来确认测试是否通过。你不应该手工对比两个不同文本文件来确认测试是否通过。如果测试不能自足验证，对失败的判断就会变得依赖主观，而运行测试也需要更长的手工操作时间。
* **及时（Timely）**测试应及时编写。单元测试应该恰好在使其通过的生产代码之前编写。如果在编写生产代码之后编写测试，你会发现生产代码难以测试。你可能会认为某些生产代码本身难以测试。你可能不会去设计可测试的代码。

# Ch10 Classes(类)

## 10.1 Class Organization(类的组织)

遵循标准的Java约定，类应该从一组变量列表开始。如果有公共静态常量，应该先出现。然后是私有静态变量，以及私有实体变量。很少会有公共变量。公共函数应跟在变量列表之后。我们喜欢把由某个公共函数调用的私有工具函数紧随在该公共函数后面。这符合了自顶向下原则，让程序读起来就像一篇报纸文章。

## 10.2 Class Should Be Small!(类应该短小)

对于函数，我们通过计算代码行数衡量大小。对于类，我们采用不同的衡量方法，计算权责（responsibility）。

1. 单一权责原则：一个类应该只有一个发生变化的原因。
2. 内聚：
   * 类应该只有少量实体变量。类中的每个方法都应该操作一个或多个这种变量。通常而言，方法操作的变量越多，就越黏聚到类上。如果一个类中的每个变量都被每个方法所使用，则该类具有最大的内聚性。
   * 一般来说，创建这种极大化内聚类是既不可取也不可能的；另一方面，我们希望内聚性保持在较高位置。内聚性高，意味着类中的方法和变量互相依赖、互相结合成一个逻辑整体。

3. 保持内聚性就会得到许多短小的类

## 10.3 Organizing for Change(为了修改而组织)

> 依赖倒置原则(DIP)：高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。

DIP认为类应当依赖于抽象而不是依赖于具体细节。

# * Ch11 Systems(系统)

## 11.1 Separate Constructing a System from Using It(将系统的构造与使用分开)

1. 分解main
2. 工厂
3. 依赖注入

## 11.2 Scaling Up(扩容)

## 11.3 Java Proxies(Java代理)

## 11.4 Pure Java AOP Frameworks(纯Java AOP框架)

## 11.5 AspectJ Aspects(AspectJ的方面)

## 11.6 Test Drive the System Architecture(测试驱动系统框架)

## 11.7 Optimize Decision Making(优化决策)

# Ch12 Emergence(迭进)

Kent Beck关于简单设计的四条规则：

1. 运行所有测试；
2. 不可重复；
3. 表达了程序员的意图；
4. 尽可能减少类和方法的数量；

以上规则按其重要程度排列。

# * Ch13 Concurrency(并发编程)

> “对象是过程的抽象。线程是调度的抽象。”   一James O Coplien

## 13.1 并发防御原则

1. 单一权责原则：分离并发相关代码与其他代码
2. 限制数据作用域：谨记数据封装；严格限制对可能被共享的数据的访问。
3. 使用数据副本
4. 线程尽可能独立：尝试将数据分解到可被独立线程（可能在不同处理器上）操作的独立子集。

# Ch14 Successive Refinement(逐步改进)

# * Ch15 JUnit Internals(JUnit内幕)

# * Ch16 Refactoring SerialDate(重构 SerialDate)

# * Ch17 Smells and Heuristics(味道与启发)

