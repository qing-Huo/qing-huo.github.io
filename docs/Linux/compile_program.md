# 编译程序

源代码的可用性是至关重要的自
由,从而使得 Linux 成为可能。整个 Linux 开发生态圈就是依赖于开发者之间的自由交流。对
于许多桌面用户来说,编译是一种失传的艺术。以前很常见,但现在,由系统发行版提供商维
护巨大的预编译的二进制仓库,准备供用户下载和使用。


为什么要编译软件呢?有两个原因:

1. 可用性。尽管系统发行版仓库中已经包含了大量的预编译程序,但是一些发行版本不可
能包含所有期望的应用。在这种情况下,得到所期望程序的唯一方式是编译程序源码。
2. 及时性。虽然一些系统发行版专门打包前沿版本的应用程序,但是很多不是。这意味着,
为了拥有一个最新版本的程序,编译是必需的。

从源码编译软件可以变得非常复杂且具有技术性;许多用户难以企及。然而,许多编译任务
是相当简单的,只涉及到几个步骤。这都取决于程序包。

## 什么是编译？

简而言之,编译就是把源码(一个由程序员编写的人类可读的程序的说明)翻译成计算机处理
器的语言的过程。

计算机处理器(或 CPU)工作在一个非常基本的水平,执行用机器语言编写的程序。这是
一种数值编码,描述非常小的操作,比如“加这个字节”、“指向内存中的这个位置”或者“复
制这个字节”。