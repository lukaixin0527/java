# 一、ETL介绍

## 1、简介

​		Kettle是一款国外开源的ETL工具，纯java编写，可以在Window、Linux、Unix上运行，绿色无需安装，数据抽取高效稳定。
​		Kettle 中文名称叫水壶，该项目的主程序员MATT 希望把各种数据放到一个壶里，然后以一种指定的格式流出。
​		Kettle这个ETL工具集，它允许你管理来自不同数据库的数据，通过提供一个图形化的用户环境来描述你想做什么，而不是你想怎么做。
​		Kettle中有两种脚本文件，transformation和job，transformation完成针对数据的基础转换，job则完成整个工作流的控制。
​		Kettle(现在已经更名为PDI，Pentaho Data Integration-Pentaho数据集成)。

​		中文官网：http://www.kettle.org.cn/  

## 2、安装

1、下载网址：https://sourceforge.net/projects/pentaho/files/

![image-20220609163857933](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220609163857933.png)

2、直接解压到指定目录即可

![kettle安装目录](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/kettle%E5%AE%89%E8%A3%85%E7%9B%AE%E5%BD%95.png)

3、配置JDK和kettle的环境变量，追加到PATH中

![image-20220507142607495](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220507142607495.png)

4、Spoon.bat为启动文件

![image-20220507142900067](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220507142900067.png)

# 二、kettle快速体验

## 把数据从CSV文件复制到Excel文件

```
1、输入框中选择，CSV文件输入
2、输出框中选择，Excel输出
3、点击CSV文件输入，设置输入文件目录，点击下面获取字段，进行设置
4、点击Excel文件输出，设置获取字段，设置字段格式。
5、点击左上角运行，选择保存输出文件的保存路径，进行转换。
```

![image-20220507143756981](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220507143756981.png)

# 三、kettle核心概念

## 1、可视化编程

​	Kettle可以被归类为可视化编程语言(Visula Programming Languages,VPL),因为Kettle可以使用图形化的方式定义复杂的ETL程序和工作流。
​	Kettle里的图就是转换和作业。
​	可视化编程一直是Kettle里的核心概念，它可以让你快速构建复杂的ETL作业和减低维护工作量。它通过隐藏很多技术细节，使IT领域更贴近于商务领域。

## 2、转换

- 转换(transaformation)是ETL解决方案中最主要的部分，它处理抽取、转换、加载各种对数据行的操作。
- 转换包含一个或多个步骤(step)，如读取文件、过滤数据行、数据清洗或将数据加载到数据库。
- 转换里的步骤通过跳(hop)来连接，跳定义一个单向通道，允许数据从一个步骤向另一个步骤流动。
- 在Kettle里，数据的单位是行，数据流就是数据行从一个步骤到另一个步骤的移动。
- 数据流有的时候也被称之为记录流。

## 3、Step步骤

步骤（控件）是转换里的基本的组成部分。

快速入门的案例中就存在两个步骤，“CSV文件输入”和“Excel输出”。
一个步骤有如下几个关键特性：
①步骤需要有一个名字，这个名字在转换范围内唯一。
②每个步骤都会读、写数据行(唯一例外是“生成记录”步骤，该步骤只写数据)。
③步骤将数据写到与之相连的一个或多个输出跳，再传送到跳的另一端的步骤。
④大多数的步骤都可以有多个输出跳。一个步骤的数据发送可以被被设置为分发和复制，分发是目标步骤轮流接收记录，复制是所有的记录被同时发送到所有的目标步骤。

## 4、Hop跳

跳就是步骤之间带箭头的连线，跳定义了步骤之间的数据通路。

跳实际上是两个步骤之间的被称之为行集的数据行缓存（行集的大小可以在转换的设置里定义）。

当行集满了，向行集写数据的步骤将停止写入，直到行集里又有了空间。
当行集空了，从行集读取数据的步骤停止读取，直到行集里又有可读的数据行。

![image-20220507144616038](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220507144616038.png)

## 5、数据行-数据类型

数据以数据行的形式沿着步骤移动。一个数据行是零到多个字段的集合，字段包含下面几种数据类型。
①String:字符类型数据
②Number:双精度浮点数。
③Integer:带符号长整型（64位）。
④BigNumber:任意精度数据。
⑤Date:带毫秒精度的日期时间值。
⑥Boolean:取值为true和false的布尔值。
⑦Binary:二进制字段可以包含图像、声音、视频及其他类型的二进制数据。

![image-20220507144712622](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220507144712622.png)

## 6、数据行-元数据

每个步骤在输出数据行时都有对字段的描述，这种描述就是数据行的元数据。
通常包含下面一些信息。
①名称：行里的字段名应用是唯一的。
②数据类型：字段的数据类型。
③格式：数据显示的方式，如Integer的#、0.00。
④长度：字符串的长度或者BigNumber类型的长度。
⑤精度：BigNumber数据类型的十进制精度。
⑥货币符号：￥
⑦小数点符号:十进制数据的小数点格式。不同文化背景下小数点符号是不同的，一般是点（.）或逗号（，）。
⑧分组符号：数值类型数据的分组符号，不同文化背景下数字里的分组符号也是不同的，一般是点（.）或逗号（，）或单引号（’）

![image-20220507144932603](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220507144932603.png)

## 7、并行

跳的这种基于行集缓存的规则允许每个步骤都是由一个独立的线程运行，这样并发程度最高。这一规则也允许数据以最小消耗内存的数据流的方式来处理。在数据仓库里，我们经常要处理大量数据，所以这种并发低消耗内存的方式也是ETL工具的核心需求。

对于kettle的转换，不可能定义一个执行顺序，因为所有步骤都以并发方式执行：当转换启动后，所有步骤都同时启动，从它们的输入跳中读取数据，并把处理过的数据写到输入跳，直到输入跳里不再有数据，就中止步骤的运行。当所有的步骤都中止了，整个转换就中止了。 （要与数据流向区分开）

如果你想要一个任务沿着指定的顺序执行，那么就要使用后面所讲的“作业”！



# 三、实战

## 1、CSV文件输入

CSV文件是一种带有固定格式的文本文件

![image-20220507145542346](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220507145542346.png)

## 2、文本文件输入

提取日志信息的数据是开发常见的操作，日志信息基本都是文本类型。