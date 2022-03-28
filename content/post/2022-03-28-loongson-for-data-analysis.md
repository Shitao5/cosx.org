---
title: "龙芯可以做数据分析吗？"
author: "邱怡轩"
date: "2022-03-28"
categories:
  - R 语言
  - 统计计算
  - 统计软件
tags:
  - 龙芯
  - CPU
  - Linux
  - RStudio
  - 编译
  - 矩阵
  - 机器学习
  - 开源
slug: loongson-for-data-analysis
---

# 见龙在田

因为众所周知的原因，近几年来，计算机处理器芯片这个高科技产品受到了社会的广泛关注。一时间，一款名为“龙芯”的 CPU 好像突然成了“全村人的希望”，不管是业内大佬还是吃瓜群众，都开始对自研芯片这个话题展开了激烈的讨论。按照官方的说法，龙芯是我国首枚拥有自主知识产权的通用高性能微处理器芯片，最初在2001年是中科院计算所下属的一个课题项目，而到了2010年又成立了龙芯中科公司，从纯科研项目走向了商业化。据不完全统计，截至2021年龙芯已经迭代完成了三大系列共20余款不同型号的产品。

然而很长一段时间以来，龙芯 CPU 一直带有一种“神秘感”，而且公众的关注度也不算高。其中的一个原因在于其前几代的产品一直没有大量进入消费领域，即使是发烧友也往往很难找到获取途径。直到2021年7月，龙芯中科发布了新一代的龙芯 3A5000，终于使得普通消费者可以在市面上较为容易地购买到基于龙芯的电脑产品。

![](https://uploads.cosx.org/2022/03/chip.jpg)
（图片来源：https://cpu.zol.com.cn/topic/7731998.html）

作为一名非硬核发烧友，在看到新一代的龙芯产品上市后，便入手了一台基于 3A5000 的笔记本电脑。事实上，3A5000 还包括了多个型号，例如我现在手头上的这台使用的就是龙芯 3A5000M 处理器，这里的 M 后缀指的是移动端，即用在笔记本电脑上。一般的 3A5000 会比我现在的这款工作频率更高，性能更强。
 
![](https://uploads.cosx.org/2022/03/3A5000.png)
（来源：龙芯 3A5000/3B5000 处理器数据手册）

于是万事俱备，终于可以一睹龙芯的真面目了。

# 芯事重重

此时你或许会疑惑，本文的标题到底想要表达什么意思。龙芯能不能做数据分析，难道不是看看它的性能就知道了吗？

故事是这样的。除去性能因素之外，关于龙芯的一个绕不开的事实是，龙芯 CPU 采用了与我们熟知的 Intel、AMD 以及苹果 M1 等主流 CPU 完全不同的架构，这意味着大家常见的 Windows 和 Mac 等操作系统并不能直接运行在龙芯上，而且许多软件也需要进行适配才能正常运行。于是，才有了许多人都曾有过的灵魂拷问：龙芯可以替代现在市面上的其他 CPU 吗？

对于这一问题，我们必须承认，不管是在性能上，还是在软件生态上，龙芯还有很长的路要走。对于龙芯实用性的讨论，从它诞生的那一刻起就没有停止过。如今在 B 站上，也可以找到大量的视频展示龙芯的使用情况。但目前看来，大家关注的焦点主要在两个方面，一是轻度办公，如文字处理、网页浏览等，二是休闲娱乐，如影音播放、电脑游戏等。而文本想探索的是一个可能略显严肃的问题：

> **龙芯可以做数据分析吗？**

之所以提出这个疑问，是因为我把自己定义为一名（伪）数据科学家，因此日常的工作和研究中会大量使用 R 和 Python 等编程工具，以及它们各自生态下的众多软件包。据我自己的了解，如今许多数据科学家大部分的时间也是在与这些数据分析工具打交道，因此如果龙芯可以顺利运行这些软件，那么它就相当于找到了一片非常广阔的应用领域。本文将着重介绍 R 语言相关的工具，并会在将来持续探索更多的可能。

# 商店和生产线

首先，要运行任何软件，龙芯都需要一个完整的操作系统予以支持。之前提到，龙芯还不能直接运行 Windows 和 Mac 等常见操作系统，但幸运的是，它支持开源的 Linux 系统内核，并且如今已经有了不少龙芯定制发行版，如龙芯官方推出的 Loongnix、
统信 UOS、龙蜥 Anolis OS 等。考虑到系统软件的完整度、使用的便利性以及视觉上的美观程度，本文选用了统信 UOS，它基于 Deepin 修改而来（同时 Deepin 又是从 Debian Linux 衍生而来），并加入了许多定制的软件。下图展示了系统界面的大概布局：

![](https://uploads.cosx.org/2022/03/os.jpg)

接下来进入正题，即尝试安装数据分析的利器 R 语言。事实上，UOS 已经提供了非常方便的安装选项，只需在终端中输入

```bash
sudo apt-get install r-base
```

就可以安装 R 语言的核心组件了。但显然，对于我这样的非硬核玩家，这种好事我自然是不会领情的。由于软件源支持的 R 只有 3.5.3 版本，已经远远落后时代，所以我选择直接从源代码编译安装 R。

在 Linux 下，安装软件主要有两种方式，一种是直接下载预编译的软件包，另一种是从源代码开始编译安装。前者类似于商店购物，直接把想要的东西打包带回家，而后者相当于你自己建了条生产线，从原料到成品，整个过程完全由你来控制。显然，后者比前者要费力得多，但许多 Linux 用户觉得后者才是 Linux 和开源软件的精髓，并且乐此不疲。于是就有了下面这张梗图：

![](https://uploads.cosx.org/2022/03/linux.jpg)
（图片来源：https://www.omgubuntu.co.uk/2019/08/linux-birthday-28-years-old）

但实际上，在 Linux 下从源代码安装 R 也并不是特别复杂，只需要进行三个主要步骤：

1. 安装必备的工具链（相当于购买基础零件和构建流水线）；
2. 配置各类选项（对最终产品的样式进行定制）；
3. 编译和安装（生产、运输）。

总体上来说，我们需要从R的官网下载源代码包并解压，然后在终端里运行如下几组命令：

```bash
sudo apt-get install gfortran libxt-dev libicu-dev libpcre2-dev libreadline-dev libcairo2-dev libjpeg62-turbo-dev libtiff-dev libcurl4-openssl-dev liblzma-dev libbz2-dev libxml2-dev

./configure --prefix=/your/installation/path/ --enable-R-profiling --enable-memory-profiling --enable-R-shlib --enable-BLAS-shlib

make && make install
```

其中 `/your/installation/path/` 指代你想要安装 R 的位置，而上面的三组命令分别对应前述的三个步骤。在经过了其他一些细节的配置环节后，我们终于成功在龙芯上运行 R 程序了！

![](https://uploads.cosx.org/2022/03/R.png)

# 道阻且长

一切似乎都很顺利？

当然，事情还远没有这么简单。除了在终端中运行，相信很大一部分 R 用户都习惯了在 RStudio 中进行 R 程序的编写；再配合各种强力 R 包，就可以解决实际中很大一部分的数据分析问题了。然而，尽管 RStudio 也是开源软件，它的编译流程却要复杂得多，而且我在尝试的过程中多次遇到了软件不支持龙芯平台的问题，例如下面这样的情形：

![](https://uploads.cosx.org/2022/03/error1.png)
![](https://uploads.cosx.org/2022/03/error2.png)

幸运的是，开源软件的一大特性就是可修改、可定制、可 DIY。在经过了阅读源代码、编写补丁和修改系统配置等一系列艰苦卓绝的努力后，终于成功将 RStudio 编译、安装和运行，从此验证了将这套数据分析系统运行在龙芯平台上是可行的。对于整个编译过程感兴趣的朋友，可以移步这个 [B 站视频（待补充）](https://bilibili.com) 观（shòu）看（kǔ）。

最后是一张 RStudio、`corrplot` 软件包与系统信息菜单的合影：

![](https://uploads.cosx.org/2022/03/rstudio.png)

# 上层建筑

前面解决了 R 与 RStudio 这两项基础设施的建设，而基于它们，我们将解锁更多更丰富的分析工具，以及随之而来的新问题。

作为 RStudio 中一项重量级的应用，R Markdown 深受数据科学家的喜爱。简单来说，R Markdown 可以用 Markdown 的语法撰写文档，同时在其中嵌入可运行的 R 代码，是 R 中生成动态报告的不二选择。既然我们已经安装好了 RStudio，那么只需要新建 R Markdown 文档，再点击 Knit 按钮，然后……然后就翻车了。

![](https://uploads.cosx.org/2022/03/rmarkdown-err.png)

翻车的原因其实也很简单。R Markdown 生成文档依赖一个名为 [Pandoc](https://pandoc.org/) 的软件，RStudio 在安装时会自动从网上下载它。然而，Pandoc 还没有提供龙芯平台的支持，所以 RStudio 下载的是 Intel 平台上的版本。由于两个平台运行软件的机制不一样，所以出现了图中的“运行格式错误”。

那么这是否说明我们无法在龙芯版的 RStudio 上使用 R Markdown 了呢？

非也。幸运的是，Pandoc 是一个单一的可执行文件，没有复杂的软件库依赖，而龙芯官方最近公开的一项黑科技正好可以解决这个难题。这一黑科技称为二进制翻译系统，作用是将其他硬件平台上的 CPU 指令翻译成龙芯支持的指令。用通俗的话来讲，就是可以直接在龙芯上运行 Intel 平台上的软件。当然，这么做并非没有代价。翻译的过程会存在一定的损耗，使得运行的效率会打折扣，这也是为什么我们要花功夫去编译 R 与 RStudio，而不是直接使用二进制翻译的原因。但因为 Pandoc 仅仅是在生成文档时才会被调用，所以即使它运行得慢一些，也基本不影响主要的工作。

![](https://uploads.cosx.org/2022/03/translate.jpg)

在当前系统中，只需要运行下面的命令安装 `lat` 这个软件包就可以启用二进制翻译了：

```
sudo apt-get install lat
```

此时再次点击 Knit 按钮，文档就顺利生成了：

![](https://uploads.cosx.org/2022/03/rmarkdown.png)

除此之外，经测试 R/RStudio 还有很多其他功能都可以在龙芯平台上顺利运行，例如 R 包开发工具、交互式应用 Shiny App、代码瓶颈分析工具 R Profiler 等等。下图是一张运行 Shiny App 的截图，其他更多应用在此就不再赘述了。希望本文能吸引更多的软硬件爱好者参与到龙芯平台的探索与研究之中。

![](https://uploads.cosx.org/2022/03/shiny.png)

# 性能为王

在解决了软件运行的难题后，龙芯就要接受更为严峻的考验——性能测试了。在如今这个数据与模型都越来越大的时代，计算平台的性能就成了做数据分析时一个非常重要的影响因素。

当然，性能的测试是一个非常复杂的问题，而本文只关心一个非常小的方面：矩阵运算。之所以关注这一项，是因为矩阵运算是许多统计和机器学习模型计算的基础，其性能很大程度上决定了跑模型所需的时间。而我也有点欺负人，不，是欺负龙了，直接给龙芯安排了一位足以碾压它的强大对手：Intel 在2021年推出的高端笔记本电脑处理器 Intel i7-11800H。

由于本文的目的只是大概展示一下龙芯的性能及其与市场高端产品的差距，因此我们牺牲一定的严谨性，而只是进行几个简单项目的测试。首先我们生成三个矩阵：`$X$` 大小为 `$2000\times 2000$`，`$Y$` 大小为 `$2000\times 1000$`，`$Z$` 为对称矩阵，大小为 `$1000\times 1000$`。测试的项目如下：

1. 矩阵乘法 `$XY$`；
2. 矩阵求逆 `$X^{-1}$`；
3. 解线性方程组 `$XB=Y$`，即计算 `$X^{-1}Y$`；
4. 对 `$Z$` 进行特征值分解；
5. 对 `$Y$` 进行奇异值分解（SVD），取10组奇异值和奇异向量。

测试的代码如下：

```r
library(bench)
library(RSpectra)
set.seed(123)

m = 2000
n = 1000
x = matrix(rnorm(m^2), m)
y = matrix(rnorm(m * n), m)
z = matrix(rnorm(n^2), n)
z = z + t(z)

bench::mark(
    x %*% y,
    solve(x),
    solve(x, y),
    eigen(z, symmetric = TRUE),
    svds(y, k = 10),
    
    check = FALSE, iterations = 30
)
```

换言之，我们将每个项目重复30次，取平均运行所需时长作为最终的结果。在默认的软件环境下，3A5000M 与 i7-11800H 的对比结果如下图所示。值得一提的是，这张图也是在龙芯笔记本上用 R 绘制的，同时还用 `showtext` 包解决了中文字体的问题。

![](https://uploads.cosx.org/2022/03/benchmark-blas.png)

需要说明的是，两台笔记本的软硬件环境差别很大，因此结果一定存在误差，但多少可以反映总体上的区别。客观上来说，这样的差距还是非常明显的。几乎对于所有的项目，i7-11800H 的运行时间差不多都只有 3A5000M 的一半，甚至更少。事实是非常残酷的，但我们也必须要面对它。

但这还不是故事的全部。

# 软硬兼施

事实上，R 的矩阵运算依赖一个名为 BLAS 的底层代数运算库，其默认的版本并没有针对各种 CPU 进行优化，而当前非常流行的一个 BLAS 的替代品是开源的 [OpenBLAS](https://www.openblas.net/)。在我为两个平台的 R 都换上优化版的 OpenBLAS 并都使用4个线程并行计算后，i7-11800H 和 3A5000M 在同样的计算任务上都获得了巨大的性能提升。

![](https://uploads.cosx.org/2022/03/benchmark-all.png)

从相对值来看，i7-11800H 的性能依然是龙芯的两倍有余，而且我在测试中限定了最多只用4个线程。实际上，这款 i7 是8核16线程，而 3A5000M 仅为4核4线程。但尽管如此，经过软件的优化后，我们看到两颗处理器在同样计算任务上的**绝对差距**大大缩小了。这说明，**即使在硬件不变的情况下，软件的进步依然大有可为**。

这一惊人的结果要归功于 OpenBLAS 强大的计算效率，其发起人和主要开发者是中科院毕业的张先轶博士。他从2013年开始就领导了 OpenBLAS 的开发，并在2021年10月的 0.3.18 版本中加入了对龙芯 3A5000 的支持。这从另一个侧面说明，与 Intel CPU 完善的生态相比，龙芯平台当前的软件还没有充分发挥硬件的实力。正如张先轶博士在一次采访中所说的那样，“软件开发者是芯片公司非常重要的资产，CPU 做出来是不够的，要让更多的软件开发者用这颗芯片才是成功。”

# 总结

那么，作为一名不懂硬件的数据科学家，有哪些事情可以帮助国产处理器芯片的发展呢？我认为至少有以下三个方面。

1. **研究和科普**。对现有的技术和产品进行研究，证明哪些可以做，哪些还需要发展。事实上，本文的出发点也正是如此，试图向大家展示龙芯在数据分析方面的潜力。就我自己体验下来，3A5000+Linux 的组合还无法成为大多数人的日常使用方案，但却已经完全可以胜任主流编程语言和统计计算的教学工作。~~愿大学的机房没有 Visual C++ 和 SAS。~~ 对于数据科学家而言，进行算法开发和轻中度的计算任务也是没有问题的。
2. **提出需求**。例如，对于统计计算和机器学习来说，矩阵运算是非常基础且重要的运算。事实上，如今的许多高端芯片，如 Nvidia RTX GPU，手机芯片中的神经网络处理器 NPU 等，都陆续加入了矩阵计算的单元。如果这些需求未来成为 CPU 的设计主流，那么尽快让芯片设计师认识到这些需求也是非常必要的。
3. **开源软件**。我们需要更多像 OpenBLAS 这样优秀的开源软件来丰富龙芯的生态，也期待未来能看到更多的开发者加入到这个生态当中。而就算退一步，像是移植 RStudio 这种工作，我也相信有其价值。至少，文本展示了 R 及其生态在龙芯平台上的巨大潜力，这意味着广大的数据科学家都是潜在的贡献者：你写的每一个 R 包，甚至每一段 R 代码，都为龙芯增加了一个应用方向。

从第一款龙芯芯片诞生到今天，已经过去了20年。但即使如此，对于 CPU 芯片这样的高端科技来说，时间和技术的积累还依然远远不够。事实上，也许绝大多数人永远也不会亲身投入到芯片设计和制造这个领域中，但我经过这一段时间的摸索和折腾，渐渐发现我们能做的也远不止关注一下业界新闻这么简单。至少在现阶段，一项非常有意义的工作就是充分发掘龙芯的平台潜力，并对数据科学常用的工具进行适配和移植。作为一名（伪）数据科学家，我惊喜地看到龙芯在数据科学领域的巨大前景，并期待着未来龙芯能承担更多的计算任务。