---
layout: post
notes: true
subtitle: "Java 8 Lambdas: Functional Programming for the Masses"
comments: false
author: "[英] Richard Warburton（王群锋 译）"
date: 2016-08-23 12:00:00

---


![](/img/notes/java/java8Lambdas/java8_lambdas.jpeg)

*   目录
{:toc }

# 第1章 简介

## 1.1 为什么需要再次修改Java

面向对象编程是对数据进行抽象，而函数式编程是对行为进行抽象。

## 1.2 什么是函数式编程

核心：在思考问题时，使用不可变值和函数，函数对一个值进行处理，映射成另一个值。

## 1.3 示例

# 第2章 Lambda表达式

Java 8的最大变化是引入了Lambda表达式——一种紧凑的、传递行为的方式。

## 2.1 第一个Lambda表达式

    button.addActionListener(event->System.out.println("button clicked"))

尽管与之前相比，Lambda表达式中的参数需要的样板代码很少，但是Java 8仍然是一种静态类型语言。为了增加可读性并迁就我们的习惯，，声明参数时也可以包括类型信息，而且有时编译器不一定能根据上下文推断出参数的类型！

## 2.2 如何辨别Lambda表达式

Lambda表达式的集中变体：

    Runnable noArguments = () -> System.out.println("Hello World");

上述所示的Lambda表达式不包含参数，使用空括号()表示没有参数。该Lambda表达式实现了Runnable接口，该接口也只有一个run方法，没有参数，且返回类型为void。

    ActionListener oneArgument = event -> System.out.println("button clicked");

上述所示的Lambda表达式包含且只包含一个参数，可省略参数的括号。

    Runnable multiStatement = () -> {
        System.out.print("Hello");
        System.out.println(" World");
    }

如上，表达式的主体不仅可以是一个表达式，而且也可以是一段代码块。

    BinaryOperator<Long> add = (x, y) -> x + y;

如上，Lambda表达式也可以表示包含多个参数的方法，这时就有必要思考怎样去阅读该Lambda表达式。这行代码并不是将两个数字相加，而是创建了一个函数，用来计算两个数字相加的结果。变量add的类型是BinaryOperator<Long>，它不是两个数字的和，而是将两个数字相加的那行代码。

    BinaryOperator<Long> addExplicit = (Long x, Long y) -> x + y;

如上，有时最好可以显式声明参数类型。

目标类型是指Lambda表达式所在上下文环境的类型。

Lambda表达式的类型依赖于上下文环境，是由编译器推断出来的。

## 2.3 引用值，而不是变量

虽然无需将变量声明为final，但在Lambda表达式中，也无法用作非终态变量，如果坚持用作非终态变量，编译器就会报错。

## 2.4 函数接口

函数接口是只有一个抽象方法的接口，用作Lambda表达式的类型。

| 接口 | 参数 | 返回类型 | 示例 |
| ---- | ---- | ---- | ---- |
| Predicate&lt;T&gt; | T | boolean | 这张唱片已经发行了吗 | 
| Consumer&lt;T&gt; | T | void | 输出一个值 |
| Function&lt;T, R&gt; | T | R | 获得Artist对象的名字 |
| Supplier&lt;T&gt; | None | T | 工厂方法 |
| UnaryOperator&lt;T&gt; | T | T | 逻辑非(!) |
| BinaryOperator&lt;T&gt; | (T, T) | T | 求两个数的乘积(*) |

## 2.5 类型推断

    Predicate<Integer> alLeast5 = x -> x > 5;
    BinaryOperator<Long> addLongs = (x, y) -> x + y;

没有泛型，代码则通不过编译

    BinaryOperator add = (x, y) -> x + y;

## 2.6 要点回顾

*   Lambda表达式是一个匿名方法，将行为像数据一样进行传递。
*   Lambda表达式的常见结构：BinaryOperator<Integer> add = (x, y) -> x + y。
*   函数接口指仅具有单个抽象方法的接口，用来表示Lambda表达式的类型。

## 2.7 练习

# 第3章 流

流使程序员得以站在更高的抽象层次上对集合进行操作。

## 3.1 从外部迭代到内部迭代

    long count = allArtists.stream()
            .filter(artist -> artist.isFrom("Londom"))
            .count();

stream是用函数式编程方式在集合类上进行复杂操作的工具

## 3.2 实现机制

*   像filter这样只描述Stream，最终不产生新集合的方法叫作**惰性求值方法**，惰性求值的返回值是Stream；
*   像count这样最终会从Stream产生值的方法叫作**及早求值方法**，及早求值返回值是另一个值或为空。 

## 3.3 常用的流操作

### 3.3.1 collect(toList())

collect(toList())方法由Stream里的值生成一个列表，是一个及早求值操作。

    List<String> collected = Stream.of("a", "b", "c")
            .collect(Collectors.toList());

### 3.3.2 map

如果有一个函数可以将一种类型的值转换成另外一种类型，map操作就可以使用该函数，将一个流中的值转换成一个新的流。

    List<String> collected = Stream.of("a", "b", "hello")
            .map(string -> string.toUpperCase())
            .collect(toList());

### 3.3.3 filter

遍历数据并检查其中的元素时，可尝试使用Stream中提供的新方法filter

    List<String> beginningWithNumbers = Stream.of("a", "1abc", "abc1")
            .filter(value -> isDigit(value.charAt(0)))
            .collect(toList());

### 3.3.4 flatMap

flatMap方法可用Stream替换值，然后将多个Stream连接成一个Stream。

    List<Integer> together = Stream.of(asList(1, 2), asList(3, 4))
            .flatMap(numbers -> numbers.stream())
            .collect(toList());

### 3.3.5 max和min

    Track shotestTrack = tracks.stream()
            .min(Comparator.comparing(track -> track.getLength()))
            .get();

Stream的min 和 max方法，返回Optional对象。通过调用get方法可以取出Optional对象中的值。

### 3.3.6 通用模式

### 3.3.7 reduce

reduce操作可以实现从一组值中生成一个值。count、min和max方法，因为常用而被纳入标准库中。这些方法都是reduce操作。

    int count = Stream.of(1, 2, 3)
            .reduce(0, (acc, element) -> acc + element);

reducer的类型是BinaryOperator。

展开reduce操作：

    BinaryOperator<Integer> accumulator = (acc, element) -> acc + element;
    int count = auumulator.apply(
            accumulator.apply(
                    accumulator.apply(0, 1),
            2),
    3);

### 3.3.8 整合操作

    Set<String> origins = album.getMusicians()
            .filter(artist -> artist.getName().startsWith("The"))
            .map(artist -> artist.getNationality())
            .collect(toSet());

通过Stream暴露集合的最大优点在于，它很好地封装了内部实现的数据结构。仅暴露一个Stream接口，用户在实际操作中无论如何使用，都不会影响内部的list或Set。

## 3.4 重构遗留代码

    albums.stream()
            .flatMap(album -> album.getTracks())
            .filter(track -> track.getLength() > 60)
            .map(track -> track.getName())
            .collect(toSet());

## 3.5 多次调用流操作

避免每一步操作都强制对函数求值

## 3.6 高阶函数

如果函数的参数列表里包含函数接口，或该函数返回一个函数接口，那么该函数就是高阶函数。

## 3.7 正确使用Lambda表达式

无论何时，将Lambda表达式传给Stream上的高阶函数，都应该尽量避免副作用。唯一的例外是forEach方法，它是一个终结方法。

## 3.8 要点回顾

*   内部迭代将更多控制权交给了集合类。
*   和Iterator类似，Stream是一种内部迭代方式。
*   将Lambda表达式和Stream上的方法结合起来，可以完成很多常见的集合操作。

## 3.9 练习

## 3.10 进阶练习

# 第4章 类库

Java 8中的另一个变化是引入了*默认方法*和接口的*静态方法*，它改变了人们认识类库的方式，接口中的方法也可以包含代码体了。

## 4.1 在代码中使用Lambda表达式

使用Lambda表达式简化日志代码

    Logger logger = new Logger();
    logger.debug(() -> "Look at this!");

启用Lambda表达式实现日志记录器

    public void debug(Supplier<String> message) {
        if (isDebugEnabled()) {
            debug(message.get());
        }
    }

## 4.2 基本类型

    IntSummaryStatistics trackLengthStats = album.getTracks()
            .mapToInt(track -> track.getLength))
            .sumaryStatistics();
    System.out.println("Max: %d, Min: %d, Avg: %f, Sum: %d",
            trackLengthStats.getMax(),
            trackLengthStats.getMin(),
            trackLengthStats.getAverage(),
            trackLengthStats.getSum());

这些统计值在所有特殊处理的Stream，如DoubleStream、LongStream中都可以得出。如无需全部的统计值，也可分别调用min、max、average或sum方法获得单个的统计值，同样，三种基本类型对应的特殊Stream也都包含这些方法。

## 4.3 重载解析

Lambda表达式作为参数时，其类型由它的目标类型推导得出，推导过程遵循如下规则：
    
*   如果只有一个可能的目标类型，由相应函数接口里的参数类型推导得出
*   如果有多个可能的目标类型，由最具体的类型推导得出
*   如果有多个可能的目标类型且最具体的类型不明确，则需人为指定类型

## 4.4 @FunctionalInterface

为了提高Stream对象可操作性而引入的各种新接口，都需要有Lambda表达式可以实现它。它们存在的意义在于将代码块作为数据打包起来。因此，它们都添加了@FunctionalInterface注释。

该注释会强制javac检查一个接口是否符合函数接口的标准。如果不符合，javac就会报错。重构代码时，使用它能很容易发现问题。

## 4.5 二进制接口的兼容性

在JDK之外实现Colleciton接口的类，需要实现新增的stream方法。为了避免这个糟糕情况，则需要在Java 8中添加新的语言特性：*默认方法*

## 4.6 默认方法

    default void forEach(Consumer<? super T> action) {
        for (T t : this) {
            action.accept(t);
        }
    }

### 默认方法和子类

*类中重写的方法胜出*

## 4.7 多重继承

接口允许多重继承，因此有可能碰到两个接口包含签名相同的默认方法的情况。如果子类不重写该方法，则javac并不明确应该继承哪个接口中的方法，因此编译器会报错。

### 三定律

1.  类胜于接口。如果在继承链中有方法体或抽象的方法声明，那么就可以忽略接口中定义的方法。
2.  子类胜于父类。如果一个接口继承了另一个接口，且两个接口都定义了一个默认方法，那么子类中定义的方法胜出。
3.  没有规则三。如果上面两条规则不适用，子类要么需要实现该方法，要么将该方法声明为抽象方法。

其中第一条规则是为了让代码向后兼容。

## 4.8 权衡

接口和抽象类之间还是存在明显的区别。接口允许多重继承，却没有成员变量；抽象类可以继承成员变量，却不能多重继承。在对问题域建模时，需要根据具体情况进行权衡，而在以前的Java中可能并不需要这样。

## 4.9 接口的静态方法

Stream是个接口，Stream.of是接口的静态方法。

Stream和其他几个子类还包含另外几个静态方法，特别是range和iterate方法提供了产生Stream的其他方式。

## 4.10 Optional

Optinal是为核心类库新设计的一个数据类型，用来替换null值。

使用Optional对象有两个目的：

*   首先，Optional对象鼓励程序员适时检查变量是否为空，以避免代码缺陷；
*   其次，它将一个类的API中可能为空的值文档化，这笔阅读实现代码要简单的多。

    Optional<String> a = Optional.of("a");
    Optional emptyOptional = Optional.empty();
    Optional alsoEmpty = Optional.ofNullable(null);
    emptyOptinal.isPresent();
    emptyOptional.orElse("b");
    emptyOptional.oeElseGet(()->"c"));

## 4.11 要点回顾

*   使用为基本类型定制的Lambda表达式和Stream，如IntStream可以显著提升系统性能。
*   默认方法是指接口中定义的包含方法体的方法，方法名有default关键字做前缀。
*   在一个值可能为空的建模情况下，使用Optional对象能替代使用null值。

## 4.12 练习

## 4.13 开放练习

# 第5章 高级集合类和收集器

## 5.1 方法引用

    artiset -> artist.getName()

用*方法引用*重写上面的Lambda表达式：

    Artist::getName

标准语法为Classname::methodName。

    （name, nationality) -> new Artist(name, nationality)

使用方法引用，上述代码可写为：

    Artist::new

创建数组：
    
    String[]::new

## 5.2 元素顺序

    List<Integer> sameOrder = numbers.stream()
            .sorted()
            .collect(toList()); 

一些操作在有序的流上开销更大，调用unordered方法消除这种顺序就能解决该问题。大多数操作都是在有序流上效率更高，比如filter、map和reduce等。

forEach方法不能保证元素是按顺序处理的。如果需要保证按顺序处理，应该使用forEachOrdered方法。  

## 5.3 使用收集器

### 5.3.1 转换成其他集合

在调用toList或者toSet方法时，不需要指定具体的类型。Stream类库在背后自动为你挑选出了合适的类型。比如，你可能希望使用TreeSet，而不是由框架在背后自动为你指定一种类型的Set，此时就可以使用toCollection，它接受一个函数作为参数，来创建集合。

    stream.collect(toCollection(TreeSet::new));

### 5.3.2 转换成值

    Function<Artist, Long> getCount = artist -> artist.getMembers().count();
    Optional<Artist> biggestGroup = artists.collect(maxBy(Comparing(getCount)));

    albums.stream().collect(averagingInt(album -> album.getTrackList().size()));

### 5.3.3 数据分块

另外一个常用的流操作是将其分解成两个集合。

有这样一个收集器partitioningBy，它接受一个流，并将其分成两部分。它使用Predicate对象判断一个元素应该属于哪个部分，并根据布尔值返回一个Map列表。

    Map<Boolean, List<Artist>> bandsAndSolo = artists.collect(partitioningBy(artist -> artist.isSolo()));

    Map<Boolean, List<Artist>> bandsAndSolo = artists.collect(partitioningBy(Artist::isSolo));

### 5.3.4 数据分组

    Map<Artist, List<Album>> albumsByArtist = albums.collect(groupingBy(album -> album.getMainMusician()));

### 5.3.5 字符串

    String result = artists.stream()
            .map(Artist::getName)
            .collect(Collectors.joining(",", "[", "]"));

Collectors.joining方法可以方便地从一个流得到一个字符串，允许用户提供分隔符、前缀和后缀

### 5.3.6 组合收集器

    Map<Artist, Long> numberOfAlbums = albums.collect(groupingBy(album -> album.getMainMuscian(), counting()));

    Map<Artist, List<String>> nameOfAlbums = albums.collect(groupingBy(Album::getMainMusician, mapping(Album::getName, toList())));

### 5.3.7 重构和定制收集器

    String result = artists.stream()
            .map(Artist::getName)
            .collect(new StringCollector(",", "[", "]"));

定义字符串收集器：
    
    public class StringCollector implements Collector<String, StringCombiner, String> {
        ...
    }

一个收集器由四部分组成。首先是一个Supplier，这是一个工厂方法，用来创建容器。

    public Supplier<StringCombiner> supplier() {
        return () -> new StringCombiner(delim, prefix, suffix);
    } 

收集器的accumulator结合之前操作的结果和当前值，生成并返回新的值。

    public BiConsumer<StringCombiner, String> accumulator() {
        return StringCombiner::add;
    }

combine方法用于合并两个容器

    public BinaryOperator<StringCombiner> combiner() {
        return StringCombiner::merge;
    }

在收集阶段，容器被combiner方法成对合并进一个容器，直到最后只剩下一个容器为止。

finisher方法，转换成最终想要的结果

    public Function<StringCombiner, String> finisher() {
        return StringCombiner::toString;
    }

### 5.3.8 对收集器的归一化处理

reducing收集器，低效，建议定制收集器

## 5.4 一些细节

用Map实现缓存的读取

    artistCache.computeIfAbsent(name, this::readArtistFromDB);

使用内部迭代遍历Map

    albumsByArtist.forEach((artist, albums) -> {
        ......
    });

## 5.5 要点回顾

*   方法引用是一种引用方法的轻量级语法，形如：ClassName::methodName。
*   收集器可用来计算流的最终值，是reduce方法的模拟。
*   Java 8 提供了收集多种容器类型的方式，同时允许用户自定义收集器。

## 5.6 练习

# 第6章 数据并行化

## 6.1 并行和并发

并发是两个任务共享时间段，并行则是两个任务在同一时间发生，比如运行在多核CPU上。如果一个程序要运行两个任务，并且只有一个CPU给它们分配了不同的时间片，那么这就是并发，而不是并行。

数据并行化：将数据分成块，为每块数据分配单独的处理单元。

当需要在大量数据上执行同样的操作时，数据并行化很管用。它将问题分解为可在多块数据上求解的形式，然后对每块数据执行运算，最后将各数据块上得到的结果汇总，从而获得最终答案。

任务并行化：线程不同，工作各异。

## 6.2 为什么并行化如此重要

## 6.3 并行化流操作

并行化操作流只需改变一个方法调用。如果已经有一个Stream对象，调用它的parallel方法就能让其拥有并行操作的能力。如果想从一个集合类创建一个流，调用parrelStream就能立即获得一个拥有并行能力的流。

    int parallelArraySum = albums.parallelStream()
            .flatMap(Album::getTracks)
            .mapToInt(Track::getLength)
            .sum();

## 6.4 模拟系统

使用蒙特卡洛模拟法并行化模拟掷骰子事件

    double fraction = 1.0 / N;
    Map<Integer, Double> parallelDiceRolls = IntStream.range(0, N)
            .parallel()
            .mapToObj(twoDiceThrows())
            .collect(groupingBy(side -> side,
                sumingDouble(n -> fraction)));

## 6.5 限制

*   reduce方法初值必须为组合函数的恒等值，拿恒等值和其他值做reduce操作时，其他值保持不变。比如求和操作，其初值必须为0。
*   reduce操作的另一个限制是组合操作必须符合结合律。这意味着只要序列的值不变，组合操作的顺序不重要。

使用parallel方法能轻易将流转换为并行流。还有一个叫sequential的方法。在要对流求值时，不能同时处于两种模式，要么是并行的，要么是串行的。如果同时调用了parallel和sequential方法，最后调用的那个方法起效。

## 6.6 性能

影响并行流性能的主要因素：

*   **数据大小**：输入数据的大小会影响并行化处理对性能的提升。
*   **源数据结构**
*   **装箱**：处理基本类型比处理装箱类型要快
*   **核的数量**
*   **单元处理开销**：花在流中每个元素身上的时间越长，并行操作带来的性能提升越明显。

    int addIntegers = values.parallelStream()
            .mapToInt(i -> i)
            .sum();

根据性能好坏，将核心类库提供的通用数据结构分成3组：

*   **性能好**：ArrayList，数组或IntStream.range，这些数据结构支持随机读取，能轻易地被任意分解。
*   **性能一般**：HashSet、TreeSet，这些数据结构不易公平地被分解，但是大多数时候分解是可能的。
*   **性能差**：有些数据结构难于分解，如LinkedList，对半分解太难了，还有Streams.iterate和BufferedReader.lines，它们长度未知，因此很难预测该在哪里分解。

在讨论流中单独操作每一块的种类时，可以分成两种不同的操作：*无状态的和有状态的*。无状态操作整个过程中不必维护状态，有状态操作则有维护状态所需的开销和限制。

如果能避开有状态，选用无状态操作，就能获得更好的并行性能。无状态操作包括map、filter和flatMap，有状态操作包括sorted、distinct和limit。

## 6.7 并行化数组操作

Java 8还引入了一些针对数组的并行操作，脱离流框架也可以使用Lambda表达式。

*   parallelPrefix：更新一个数组，将每一个元素替换为当前元素和前驱元素的和，这里的"和"是一个宽泛的概念，它不必是加法，可以是任意一个BinaryOperator。
*   parallelSetAll：使用Lambda表达式更新数组元素
*   parallelSort：并行化对数组元素排序

使用并行化数组操作初始化数组

    Arrays.parallelSetAll(values, i -> i);

计算简单滑动平均数

    Arrays.parallelPrefix(sums, Double::sum);
    int start = n - 1;
    double[] simpleMovingAverage = IntStream.range(start, sums.length)
            .mapToDouble(i -> {
                double prefix = i == start ? 0 : sums[i - n];
                return (sums[i] - prefix) / n;
            })
            .toArray();

## 6.8 要点回顾

*   数据并行化是把工作拆分，同时在多核CPU上执行的方式。
*   如果使用流编写代码，可通过调用parallel或者parallelStream方法实现数据并行化操作。
*   影响性能的五要素是：数据大小、源数据结构、值是否装箱、可用的CPU核数量，以及处理每个元素所花的时间。

## 6.9 练习
