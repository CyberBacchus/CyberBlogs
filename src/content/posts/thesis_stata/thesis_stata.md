---
title: 我如何完成毕业论文：基于Stata的面板数据处理技术流程
published: 2024-05-17
description: '记录自己使用Stata完成毕业论文实证部分的心得'
image: './cover.png'
tags: [Stata, Panel data, Fixed effect]
category: 'Guides'
draft: false 
---

:::note[前言]
本文意在总结与回顾我使用Stata处理与分析面板数据完成毕业论文的技术心得，方便自己日后遇到相似问题时参考
:::

:::tip
我的毕业论文使用了[Markstat](https://grodri.github.io/markstat/)包实现了在Markdown中写Stata代码的效果，排版上用了MS Word。不过我认为自己之后不会再使用它们了，之后会[在Jupyter Notebook中使用Stata](https://www.stata.com/features/overview/jupyter-notebooks/)，排版会用Typst和LaTeX。
Jupyer中可用的[Stata Kernel](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels)参考。
:::
## Why Stata?

简而言之，Stata的面板数据设定是内生的，无论是用R去`library`个什么包，还是用Python去`import`些什么，都没有`xtset`直观简洁，在最广泛使用的面板数据分析方法上Stata有专业优势。

同时，Stata面板数据分析有比较充足的互联网社区讨论与教学资源，我的许多方法都来自对Bilibili的[silencedream](https://space.bilibili.com/9840842)、[拿铁一定要加冰](https://space.bilibili.com/40545247)、[墨寒轩主人](https://space.bilibili.com/431395)等Up主的视频与对[连享会](https://www.lianxh.cn/)的文章的学习。

## 面板数据模型识别

### 模型识别分支

面板数据识别的有两个分支，首先是判断是否存在个体效应（能否用混合回归？），如果存在个体效应（也就是截距项），这个截距是依个体不同的（固定效应模型），还是均一的，也就意味着个体差异全部来自随机（随机效应模型）？

- 面板数据模型识别
    - 不存在个体效应 → 混合回归
    - 存在个体效应
        - 截距依个体不同 → 固定效应 FE
        - 不同个体截距相同  → 随机效应 RE

固定效应模型将个体效应视为代估计参数，而随机效应模型将其视为随机干扰项的一部分。

一般来说，个体效应基本都存在，Hausman检验也基本都会判断为固定效应模型，但还是进行模型识别这一步为好。

### 代码实现

在技术实现上，使用`xtreg`命令分别估计固定效应模型与随机效应模型，随后进行Hausman检验。
```stata
xtreg 被解释变量 解释变量们, fe
est store fe

xtreg 被解释变量 解释变量们, re
est store re
xttest0

hausman fe re 
```
其实这个Hausman检验就是在对比FE和RE估计参数是否相同，如果不同认为固定效应模型的参数估计结果一致，而随机效应的不一致。

### 解读实证结果

观察结果`fe`，最后一行`F test that all u_i=0: F(3953, 25803) = 4.51                 Prob > F = 0.0000`说明个体效应显著

使用`xttest0`检验随机效应，由`Prob > chibar2 =   0.0000`知随机效应显著

Hausman检验的结果显示：`chi2(5) = (b-B)'[(V_b-V_B)^(-1)](b-B) = 3751.96`，`Prob > chi2 =  0.0000`，可知应当拒绝原假设，认为固定效应模型更优。


更多细节可以阅读：[Stata: 面板数据模型一文读懂](https://www.lianxh.cn/news/bf27906144b4e.html)

## 控制变量选取：gsreg

在控制变量的选取上，我相信不少同学都会选择参考和自己主题相似的文献进行变量选取。但全局搜索回归gsreg给出了一个更合适的、更科学的变量择优方式，可以使我们面对导师或答辩的疑问会更游刃有余。

关于gsreg可以参考[gsreg：自动模型设定和变量筛选
](https://www.lianxh.cn/news/61ae7a22439cf.html)

在实际操作上，我主要参考了[拿铁一定要加冰](https://space.bilibili.com/40545247) 的[【Stata】gsreg搭配reghdfe的高维固定效应筛选最优控制变量](https://www.bilibili.com/video/BV1cY4y167a9)

代码为：
```stata
gsreg 因变量 候选变量们,fixvar(解释变量) replace cmdest(reghdfe) cmdoptions(absorb(i.固定项) vce(robust))
```
这里的`cmdest`选择了`reghdfe`进行回归，通过`cmdoptions`指定了如何调用`reghdfe`,例如选择固定目标和使用稳健标准误等。这里应当根据自己实际使用的固定效应命令和options进行调整。


## 固定效应模型：reghdfe

`reghdfe`是Stata的一个高性能估计固定效应模型的外部命令，需要通过`ssc`安装：

```stata
ssc install gtools, replace  // 依赖gtools
ssc install reghdfe, replace // 安装最新版命令
```
具体命令语法：

```stata
reghdfe 被解释变量 解释变量们 [if] [in], absorb(固定效应们)
```

通过这一命令，可以实现基准回归与异质性分析（只需通过`if`指定分类变量），例如：

```stata
## 是否国有企业
quietly reghdfe 被解释变量 解释变量 if 分类变量==1, absorb(固定项) vce(robust)
est store soe1
quietly reghdfe 被解释变量 解释变量 if 分类变量==0, absorb(固定项) vce(robust)
est store soe0
lxhreg soe1 soe0
```

也可以通过`if`指定不包含的样本（如剔除金融危机期间、剔除直辖市）进行基础的通稳健性检验。

关于`reghdfe`可以参考 [reghdfe：多维面板固定效应估计](https://www.lianxh.cn/details/156.html)

## 工具变量法

工具变量法目的是处理内生性，内生性即解释变量与随机误差项存在相关性，会导致得到有偏估计和错误的政策建议。

关于内生性及其处理，推荐阅读[内生性！内生性！解决方法大集合](https://www.lianxh.cn/details/579.html)

我进行工具变量法的代码为：

```stata
ivregress 2sls 被解释变量 控制变量们 i.固定项1 i.固定项2 (解释变量 = 工具变量),r first
est store iv_time
estat endog
estat firststage, all
```
:::note
我对工具变量法的理解其实存在许多不足，这里心得暂时留空，之后等我学得更深入了可以补上。建议阅读[IV专题: 内生性检验与过度识别检验](https://www.lianxh.cn/news/cbb06931b699e.html)
:::


## 中介效应检验：Sobel + Bootstrap
中介效应指的是自变量X通过影响中介变量M来间接影响因变量Y的过程。中介效应的实现大致有三步法中介效应、sobel检验、bootstrap检验等方法。

:::warning
虽然我们经常能在中文经济学论文中发现中介效应部分，但中介效应在经济学中的使用其实存在较多争议，对这一方法请谨慎使用。关于其存在的问题请看 [中介效应分析：三段式中介效应模型真的适用于经济学研究吗？](https://www.lianxh.cn/news/2245bd027e7bd.html)
:::

我在自己的论文中，按这样的方式实现了中介效应的机制分析：

```stata
### sobel检验
sgmediation2 GTFP,mv(lnRDMV) iv(CDT) cv(FirmAge Size Lev ROA ATO PTN Dual i.Province i.Industry)
sgmediation2 GTFP,mv(HiEduRatio) iv(CDT) cv(FirmAge Size Lev ROA ATO PTN Dual i.Province i.Industry)
sgmediation2 GTFP,mv(FIXED) iv(CDT) cv(FirmAge Size Lev ROA ATO PTN i.Province i.Industry)

### bootstrap检验
bootstrap r(ind_eff) r(dir_eff), reps(1000): sgmediation2 GTFP,mv(lnRDMV) iv(CDT) cv(FirmAge Size Lev ROA ATO PTN Dual i.Province i.Industry)
bootstrap r(ind_eff) r(dir_eff), reps(1000): sgmediation2 GTFP,mv(HiEduRatio) iv(CDT) cv(FirmAge Size Lev ROA ATO PTN Dual i.Province i.Industry)
bootstrap r(ind_eff) r(dir_eff), reps(1000): sgmediation2 GTFP,mv(FIXED) iv(CDT) cv(FirmAge Size Lev ROA ATO PTN Dual i.Province i.Industry)
```

其中，`sgmediation2`命令（需要通过ssc下载）实现了Sobel、Aroian、Goodman检验，而Bootstrap检验放宽了三步法和Sobel检验正态假设，进一步提升了结论的稳健性。

关于`sgmediation2`命令，请参考 [Stata：中介效应分析新命令-sgmediation2
](https://www.lianxh.cn/details/981.html)

## 完成分析了，然后呢？
:::important
虽然这部分出于逻辑原因放在最后，但相比前面的简单流水账，这部分才是我认为最重要的。
:::

如何展示实证结果？也就是说，我们实证做出来的表格如何放到MS Word或LaTeX中？这是个问题。最广泛使用的命令可以参见[Stata：一文搞定论文表1——基本统计量列表](https://www.lianxh.cn/details/22.html)，我最开始使用的命令是`esttab`，但最简单合适的命令莫过于[Stata：毕业论文大礼包 C——新版 esttab
](https://www.lianxh.cn/details/263.html)这篇文章提供的lxh系列命令。我最常使用的是其中的`lxhsum`（展示和输出描述性统计结果）和`lxhreg`(展示和输出回归结果)

:::caution
该文中提到的通过`lxhinstall`安装的方法已不可用，需要下载文章附件中的ado文件并移动到Stata的相应位置，以在程序中调用lxh系列命令。具体可以参考[STATA实证分析 结果导出命令分享(lxhreg lxhsum)](https://www.bilibili.com/video/BV1n14y1m7qo)
:::

为什么要用lxh系列命令？因为它是默认调优、包装到适合直接输出的`esttab`，默认就是合适的格式、保留三位小数，语法简洁、赏心悦目。具体使用如下：

```stata
lxhsum 描述统计变量 using 文件地址.tex,replace t // 使用LaTeX
lxhsum 描述统计变量 using 文件地址.rtf,replace t // 使用Word插入rtf

est store 需要输出的回归结果
lxhreg 回归结果们 using 文件地址.tex,replace t // 使用LaTeX
lxhreg 回归结果们 using 文件地址.rtf,replace t // 使用Word插入rtf
```
需要注意的是，这里的`回归结果们`可以用`*`简写，例如回归结果都`est store`为r开头，就可以表述成`r*`，并且直接使用`*`默认调用全部回归结果。

