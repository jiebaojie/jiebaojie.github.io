---
layout: post
notes: true
subtitle: "【技术博客】从JVM说起到初探Scala应用实践"
comments: false
author: "neoremind"
date: 2016-09-25 12:00:00

---


原文：[http://neoremind.com/2016/08/%E4%BB%8Ejvm%E8%AF%B4%E8%B5%B7%E5%88%B0%E5%88%9D%E6%8E%A2scala%E5%BA%94%E7%94%A8%E5%AE%9E%E8%B7%B5/](http://neoremind.com/2016/08/%E4%BB%8Ejvm%E8%AF%B4%E8%B5%B7%E5%88%B0%E5%88%9D%E6%8E%A2scala%E5%BA%94%E7%94%A8%E5%AE%9E%E8%B7%B5/)

PPT链接：[请点此](http://www.docin.com/p-1700617388.html)

*   目录
{:toc }

# 先说JVM

JVM可以是规范，也可以是实现

## JVM指令集、字节码、机器码、JIT的理解

*   JVM带有自己的指令集（instruction set），行话叫作IR（Intermediate Representation，中间表示）。
*   指令集叫做字节码（bytecode），JVM规范中已定义，需符合。
*   解释（Interpretation）和(JIT(Just-in-time)complilation)是两种执行字节码的方式。
*   解释开始执行快，但是执行中慢；JIT执行快，但是生成native code耗时。
*   Native code是各个平台自己的指令集，主流平台包括x86（桌面/服务器），ARM（移动设备）。
*   JVM可以计算运行的字节码，频繁调用的在后台默默的用JIT编译下（注意这里是需要消耗CPU资源的），然后替换掉字节码，从而可以加速执行。
*   跨平台的是编译后的bytecode还有Class文件，所以你还是需要下载指定平台的JVM。

程序编译和解释有什么区别？

*   解释过程依次取出字节码指令操作，就是改变了Java运行时内存而已。
*   JIT编译只不过把bytecode翻译成了native code（机器码）来直接执行而已。

## 怎么写一个JVM出来

实现一台JVM，只需正确读取Class文件之中每一条字节码指令，执行指令所蕴含的操作。

    do {
        自动计算PC寄存器;
        从PC寄存器的位置取出操作码(opcode);
        if (存在操作数) {
            从字节码流中取出操作数(oprands);
        }
        执行操作码定义的操作;
    } while(处理下一次循环);

## Class文件格式

javap -verbose Test。由JDK提供的工具可以方便的查看Class文件内容。

## JVM编译

编译流程：

*   词法分析 -> 单词（token）流
*   语法分析 -> 语法树/抽象语法树
*   语义分析 -> 标注了属性的抽象语法树
*   代码生成 -> 目标代码

执行流程：

*   操作系统/硬件 -> 执行结果

## 操作字节码的工具

字节码操作类库：

*   ASM（最强大，性能最好，但是需要熟悉JVM指令集，使用难度大）
*   BECL（已过时）
*   Javassit（API友好，性能佳，更新及时，文档丰富，目前最流行）
*   CGLib（2002年救出来的，性能不错，SpringAOP默认代理，更新不及时）
*   Byte buddy（后起之秀，宣称测试性能仅次于ASM）

字节码织入修改：

*   Agent, java.lang.instrumentation，自Java 1.5版引入，可以不用修改源代码，操作字节码，使用ClassFileTransformer修改待加载的字节码，可以看做是一种hook。
*   适配框架，为出入口生成准备织入（weave）的字节码，以jar包形式存在。
*   使用asm的API在待织入的框架method周围，加入jar包内的字节码，记录调用参数、耗时、异常栈等信息，最终发送到message collector中，随后由logstash导入ES。

# 再说Scala

## Helloworld

    object HelloWorld {

        def main(args : Array[String]) {

            println("hello, world!")
        }
    }

## 一切皆对象，函数是一等公民

*   Every value is an object in Scala and functions are first class values
*   函数都有返回值，最后一行就是返回值，不需要return
*   函数具有默认参数和带名参数
*   对于一个函数，你可以：
    *   调用它
    *   传递它，存放到变量中，或者作为参数传递给另外一个函数

## 变量申明和函数

Declaring variables:

    var x : Int = 7
    var x = 7       // type inferred
    val y = "hi"    // read-only

Functions:

    def square(x : Int): Int = x * x
    def square(x : Int): Int = { 
        x * x 
    }
    def announce(text : String) = {
        println(text)
    }

## 高阶函数（High-order function）

高阶函数的理解：一个函数接收另一个函数作为参数，这种函数就称之为高阶函数

高阶函数是可以组合使用的。

## 函数式编程思想

*   高阶函数解放思想，站在更高的抽象层次考虑问题
*   权责让渡，把单调的任务托付给语言和运行时，例如GC。
*   全新的编程范式：
    *   OO：程序是一系列改变状态的命令。面向对象的封装、作用域、可见性控制和感知状态，维系状态所需的众多类和交互派生出了"不确定因素"，实现粗粒度的复用
    *   FP：函数式推崇消除变量，实现可不变，无副作用，少数核心数据结构，大量的高阶函数调整底层的运转方式，不要让类和方法耦合在一起，依靠一种零件组装的方式组织抽象，减少不确定因素，是一种细粒度的重用。
*   设计模式是为了弥补语言上的缺陷？函数式来拯救
    *   命令模式在函数式编程下，"Command"不用再封装一个类，而是直接作为一个函数存在。
    *   模板模式，不用继承了，直接传递一个函数即可。
    *   ...

## 推荐书籍

*   SCALA for the Impatient
*   Java 8 函数式编程
*   函数式编程思维
*   Scala与Clojure函数式编程模式——Java虚拟机高效编程

# 最后聊聊Spark

## Spark简介

UC Berkeley AMP lab所开源的类Hadoop MapReduce的通用的并行计算框架[http://spark.apache.org](http://spark.apache.org)

核心概念：RDD（Resilient Distribued Dataset）

RDD上支持两种操作：比MR更加通用的计算模型

*   actions：在数据集上计算后返回值
*   transformations：转换，从一个RDD转换为另外一个RDD

RDD 来源：

*   FS、HDFS
*   其他RDD

在ML、DM等内存迭代多的场景下有性能优势

RDD支持故障恢复：

*   Checkpoint，持久化中间计算结果
*   Lineage，重放计算过程

RDD的transformation是惰性求值的，多个RDD组成了一个有向无环图（DAG）