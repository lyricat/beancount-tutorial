---
title: 握着你的手写最简单的 Beancount 账本
brief: 介绍 Beancount 的安装和最简单的账本组成。
date: 2019-01-10
tags: ["Beancount", "复式记账", "Beancount 教程"]
---

在[上一篇](beancount-tutorial-0)中，我提到了为什么要记账、使用复式记账的好处，以及作为一个程序员，对财务有更高要求时，要使用 Beancount。

本篇将从 Beancount 的安装开始，介绍如何编写一个最简单的 Beancount 账本。

---

## 安装 Beancount

首先你需要安装有 Python。如果没有的话，推荐去下载 [Anaconda](https://www.anaconda.com/downloads) 或者 [Miniconda](https://conda.io/miniconda.html)，里面会内置 Python。

接下来就可以用 pip 安装 Beancount 了：

```bash
$ pip install beancount
```

安装完成后，你应该可以在命令行中运行 bean- 开头的命令。比如使用 `bean-example` 会生成一个 beancount 账本范例。

## 开始记账

Beancount 账本由纯文本文件构成。你可以使用 `include "文件名"` 语法来自由组织账本文件。这里我提供三个文件组成的最简单账本（参见 [Github mini-bean 目录](https://github.com/lyricat/beancount-tutorial/)）：

### accounts.bean

accounts.bean：用来管理不同的账户，在此我开了三个账户，分别是现金账户 `Assets:Cash`，用于我的现金资产；打车账户 `Expenses:Traffic:Taxi`，用于记录打车消费；权益启动账户 `Equity:OpenBalance`，因为复式记账每一笔帐都是平的，钱不能凭空出现，因此使用这个账户来设置记账日开始自己的基准资产。

```beancount
1990-01-01 open Assets:Cash
1990-01-01 open Expenses:Traffic:Taxi
1990-01-01 open Equity:OpenBalance
```

开账户的语法为 `日期 open 账户名`。其中账户名由若干个首字母大写的单词和冒号`:`组成，`:`表示账户的父子关系。例如账户 `Expenses:Traffic:Taxi` 在语义上的含义为：消费 -> 交通消费 -> 出租消费。

在实践上，不建议同时对父子关系所属的账户记账，而是统一记录在某一个层级上。例如，在出租和公交上都有消费，那么应该创建两个帐号分别记录：

```beancount
1990-01-01 open Expenses:Traffic:Taxi
1990-01-01 open Expenses:Traffic:Public
```

而不是用含糊的 `Expenses:Traffic`来记公交的支出

```beancount
1990-01-01 open Expenses:Traffic:Taxi
1990-01-01 open Expenses:Traffic
```

### 2018.bean 

2018.bean：每年的账本，文件名就是日期。这里指代 2018 年 的账本

```beancount
2018-01-26 * "小桔科技" "打车" #Didi
    Assets:Cash			       -15.00 CNY
    Expenses:Traffic:Taxi       15.00 CNY

2018-01-25 * "初始化"
    Equity:OpenBalance			
    Assets:Cash			        100.00 CNY
```

以上是使用 beancount 记账的基本格式。每一笔交易可以分为至少 3 行。每一笔交易的第一行都以时间开头，格式是固定的：

```beancount
时间 是否核实 交易方 描述 #标签 ^链接
```

其中交易方、标签和链接是可选参数。例如上文的两笔交易分别解释如下：

**第一笔**

- 时间：2018年1月25日
- 核实：`*`表示已核实
- 描述：账本初始化
- 涉及账户：
  - `Assets:Cash`：现金从权益账户转移到本资产账户，新增 100 元
  - `Equity:OpenBalance`：这笔交易只涉及两个账户且资产账户已经指定变动金额，因此本账户会减去 100 元，可以省略不记，Beancount 会自动配平。

**第二笔**

- 时间：2018年1月26日
- 核实：`*`表示已核实
- 交易方：小桔科技
- 描述：打车
- 标签：#Didi 表示使用滴滴快车来打车
- 涉及账户：
  - `Assets:Cash`：从现金中减去 15.00 块
  - `Expenses:Traffic:Taxi` ：在打车这个消费账户花了 15.00 块

### main.bean

main.bean：主入口文件，使用 include 语句来包含 accounts.bean 和 2018.bean

```beancount
; 设定账本标题
option "title" "我的账本"

; 设定账本主货币
option "operating_currency" "CNY"

; 所有使用中的账户都写在 accounts.bean
include "accounts.bean"

; 每年的账本
include "2018.bean"
```

在 main.bean 中，我首先进行了一些设置。使用 `options` 语句，设定 beancount 的基础行为。例如，我设定了标题，还设定了账本使用的主货币为人民币 `CNY`。

最后，使用 `include` 语法将刚才我们提到账户定义 `accounts.bean`和具体到月份的账本文件 `2018.bean`包含进来。

