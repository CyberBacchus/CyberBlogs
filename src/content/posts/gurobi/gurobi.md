---
title: 使用Gurobipy优雅地构造数学优化模型
published: 2024-05-15
description: '学习Gurobi Python Interface解决数学规划问题的代码心得'
image: './cover.png'
tags: ["Mathematical Optimization"]
category: 'Guides'
draft: false 
---

> 本文是我在使用Gurobipy实现《数学建模算法与应用（第三版）》优化类例题习题的过程中总结的Gurobipy代码风格和建议，Github项目如下：

::github{repo="CyberBacchus/optimization_using_gurobi"}

## Why [Gurobi](https://www.gurobi.com/)?
#### 简洁易懂、从问题出发的 Python Interface
使用 Gurobipy 构建数学规划模型的过程非常符合直觉：你可以想到哪里构建到哪里，眼睛读到哪里就先补上这一部分，无论它是决策变量、限制条件、目标函数还是其他什么要素。其他数学优化求解器（如[杉树求解器copt](https://www.shanshu.ai/solver)）也使用了基本一致的语法，这也为代码的迁移提供了方便。

#### 强大的性能：

虽然我现在写写习题用不到什么性能，但未来上手比赛题目也许就用得上了。关于求解器的性能Benchmark可以看:
- [BENCHMARKS FOR OPTIMIZATION SOFTWARE](https://plato.asu.edu/bench.html)
- [Visualizations of Mittelmann benchmarks](https://mattmilten.github.io/mittelmann-plots/) (这是上面那个的可视化)

## Gurobi的导入

我在编写gurobipy代码早期一般用`from gurobi import *`, 之后改成了官网代码的标准方式，即：
```python
import gurobipy as gp
from gurobipy import GRB
```
## 善用 `gp.multidict` 表述问题背景

最开始编写`gurobipy`代码时，我主要参考了中文手册的基础例子和[《运筹优化常用模型、算法及案例实战——Python+Java实现》](https://www.tup.com.cn/Wap/tsxqy.aspx?id=09109001)的一些例子,对于通过批量操作构造模型的认识停留在嵌套for循环的层面。这个时期我的代码是这样的：

```python
# 通过循环给出决策变量
x = [[[] for i in range(4)] for j in range(4)]

for i in range(4):
    for j in range(4):
        x[i][j] = m.addVar(name=f"x_{i}_{j}")
```

然而通过阅读和尝试Gurobi官方提供的例子:

::github{repo="Gurobi/modeling-examples"}

我逐渐熟悉了gurobipy的代码风格，发现这样的代码是完全没有必要的，可以用
```python
x = m.addMVar((4, 4), name = "x")
```
或使用`gp.multidict`直接表述出变量的`tupledict`，从而通过
```python
x = m.addVars(tupledict_x, name = "x")
```
添加更符合问题背景的决策变量。以《数学建模算法与应用（第三版）》例题2.6为例，要表述这样一个指派问题，可以通过`gp.multidict`表述问题如下：
```python
combos, costs = gp.multidict(
    {
        ("A", "1"): 15,
        ("A", "2"): 13.8,
        ("A", "3"): 12.5,
        ("A", "4"): 11,
        ("A", "5"): 14.3,
        ("B", "1"): 14.5,
        ("B", "2"): 14,
        ("B", "3"): 13.2,
        ("B", "4"): 10.5,
        ("B", "5"): 15,
        ("C", "1"): 13.8,
        ("C", "2"): 13,
        ("C", "3"): 12.8,
        ("C", "4"): 11.3,
        ("C", "5"): 14.6,
        ("D", "1"): 14.7,
        ("D", "2"): 13.6,
        ("D", "3"): 13,
        ("D", "4"): 11.6,
        ("D", "5"): 14,
    }
)
```
从而，可以通过如下方式直接设置决策变量
```python
x = m.addVars(combos, vtype=GRB.BINARY, name="x")
```
如此基于`tupledict`的决策变量可以提升模型输出的lp文件的可读性：
```lp
\ LP format - for model browsing. Use MPS format to capture full model detail.
Minimize
  15 x[A,1] + 13.8 x[A,2] + 12.5 x[A,3] + 11 x[A,4] + 14.3 x[A,5]
   + 14.5 x[B,1] + 14 x[B,2] + 13.2 x[B,3] + 10.5 x[B,4] + 15 x[B,5]
   + 13.8 x[C,1] + 13 x[C,2] + 12.8 x[C,3] + 11.3 x[C,4] + 14.6 x[C,5]
   + 14.7 x[D,1] + 13.6 x[D,2] + 13 x[D,3] + 11.6 x[D,4] + 14 x[D,5]
Subject To
 公司限制[A]: x[A,1] + x[A,2] + x[A,3] + x[A,4] + x[A,5] <= 2
 公司限制[B]: x[B,1] + x[B,2] + x[B,3] + x[B,4] + x[B,5] <= 2
 公司限制[C]: x[C,1] + x[C,2] + x[C,3] + x[C,4] + x[C,5] <= 2
 公司限制[D]: x[D,1] + x[D,2] + x[D,3] + x[D,4] + x[D,5] <= 2
 门店限制[1]: x[A,1] + x[B,1] + x[C,1] + x[D,1] = 1
 门店限制[2]: x[A,2] + x[B,2] + x[C,2] + x[D,2] = 1
 门店限制[3]: x[A,3] + x[B,3] + x[C,3] + x[D,3] = 1
 门店限制[4]: x[A,4] + x[B,4] + x[C,4] + x[D,4] = 1
 门店限制[5]: x[A,5] + x[B,5] + x[C,5] + x[D,5] = 1
Bounds
Binaries
 x[A,1] x[A,2] x[A,3] x[A,4] x[A,5] x[B,1] x[B,2] x[B,3] x[B,4] x[B,5]
 x[C,1] x[C,2] x[C,3] x[C,4] x[C,5] x[D,1] x[D,2] x[D,3] x[D,4] x[D,5]
End
```

## 使用矩阵-矢量计算得到表达式
要获得优化对象函数，往往需要进行矩阵乘法或矢量运算，问题规模一大，手动构建表达式会异常耗时耗力。更好的解决方法是使用`x.prod()`进行矩阵乘法，使用`x.sum()`进行加总，或者更多的名称中带M（即Matrix）的Model方法，从而构造所需的表达式。延续上例：
```python
m.setObjective(x.prod(costs))
```
这样一个简洁的表达就实现了上文lp文件中冗长的Minimize部分。

## 使用Generator批量添加Constrains

`m.addConstrs`接受Generator对象，下例中使用这种方式生成了针对各个公司和门店的Constrains
```python
m.addConstrs((x.sum("*", shop) == 1 for shop in shops), name="门店限制")
m.addConstrs((x.sum(company, "*") <= 2 for company in companies), name="公司限制")
```
在向模型中添加元素时，我总是建议指定`name`，这可以方便后续输出lp文件对模型进行检查，也能提升代码的可读性。

## 获得模型结果

一般使用`m.write("文件名.lp")`输出并检查lp文件，无问题后使用`m.optimize()`运行求解，并通过以下方式获取目标函数结果和变量取值：

```python
m.optimize()

print(f"Objective value: {m.ObjVal}")
print("Optimal solution:")
for i in m.getVars():
    if i.X > 1e-6:
        print(f"{i.VarName} = {i.X}", end="\n")
```
:::important[结语]
我的gurobipy学习才刚刚起步，这些只是我的不成熟的总结，之后肯定会发现更多技巧并认识到我现在使用的方法存在的不足。待更新……
:::
