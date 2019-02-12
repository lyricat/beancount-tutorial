---
title: 握着你的手写最简单的 Beancount 账本
brief: 从几个不同应用场景，基于实际使用案例，介绍 Beancount 的用法。
date: 2019-01-10
tags: ["Beancount", "复式记账", "Beancount 教程"]
---

在上一篇中，我提到了为什么要记账、使用复式记账的好处，以及作为一个程序员，对财务有更高要求时，要使用 Beancount。

这些都是铺垫，接下来的文章我将会从几个不同应用场景，基于实际使用案例，介绍 Beancount 的用法。

---

## 安装 Beancount

首先你需要安装有 Python。如果没有的话，推荐去下载 [Anaconda](https://www.anaconda.com/downloads) 或者 [Miniconda](https://conda.io/miniconda.html)，里面会内置 Python。

接下来就可以用 pip 安装 Beancount 了：

```bash
$ pip install beancount
```

安装完成后，你应该可以在命令行中运行 bean- 开头的命令。比如使用 `bean-example` 会生成一个 beancount 账本范例。

## 开始记账

Beancount 账本由纯文本文件构成。你可以使用 `include "文件名"` 语法来自由组织账本文件。这里我提供三个文件组成的最简单账本：

### accounts.bean

accounts.bean：用来管理不同的账户，在此我开了三个账户，分别是现金账户 `Assets:Cash`，用于我的现金资产；打车账户 `Expenses:Traffic:Taxi`，用于记录打车消费；权益启动账户 `Equity:OpenBalance`，因为复式记账每一笔帐都是平的，钱不能凭空出现，因此使用这个账户来设置记账日开始自己的基准资产。

```beancount
1990-01-01 open Assets:Cash
1990-01-01 open Expenses:Traffic:Taxi
1990-01-01 open Equity:OpenBalance
```

开账户的语法为 `日期 open 账户名`。其中账户名由若干个首字母大写的单词和冒号`:`组成，`:`表示账户的父子关系。例如账户 `Expenses:Traffic:Taxi` 在语义上的含义为：消费 -> 交通消费 -> 出租消费。

在实践上，不建议同时对父子关系所属的账户记账，而是统一记录在某一个层级上。例如，在出租和公交上都有消费，那么应该创建两个帐号分别记录：

```
1990-01-01 open Expenses:Traffic:Taxi
1990-01-01 open Expenses:Traffic:Public
```

而不是用含糊的 `Expenses:Traffic`来记公交的支出

```1990-01-01 open Expenses:Traffic:Didi
1990-01-01 open Expenses:Traffic:Taxi
1990-01-01 open Expenses:Traffic
```

### 2018.bean 

2018.bean：每年的账本，文件名就是日期。这里指代 2018 年 的账本

```
2018-01-26 * "小桔科技" "打车" #Didi
    Assets:Cash			       -15.00 CNY
    Expenses:Traffic:Taxi       15.00 CNY

2018-01-25 * "初始化"
    Equity:OpenBalance			
    Assets:Cash			        100.00 CNY
```

以上是使用 beancount 记账的基本格式。每一笔交易可以分为至少 3 行。每一笔交易的第一行都以时间开头，格式是固定的：

```
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
; 设定账本标题，会出现在 fava 的左上角
option "title" "我的账本"

; 设定账本主货币
option "operating_currency" "CNY"
option "operating_currency" "USD"
option "operating_currency" "BTC"

; 所有使用中的账户都写在 accounts.bean
include "accounts.bean"

; 每年的账本
include "2018.bean"
```

在 main.bean 中，我首先进行了一些设置。使用 `options` 语句，设定 beancount 的基础行为。例如，我设定了标题，还设定了账本使用的主货币为人民币 `CNY`，美元 `USD` 和比特币 `BTC`，这是因为我交易美国股票和数字货币。

最后，使用 `include` 语法将刚才我们提到账户定义 `accounts.bean`和具体到月份的账本文件 `2018.bean`包含进来。

## 使用 Beancount 查询账本

Beancount 的命令都以 bean-* 开头。例如，我们可以在终端中使用 `bean-report` 来查询当前账户的净值。查询一下刚才我们建立的测试账本：

```bash
$ bean-report main.bean networth
Currency  Net Worth
--------  ---------
CNY           85.00
--------  ---------
$
```

资产净值为 85 块。

## 使用 Fava 展示账本

虽然 beancount 的命令提供了 beancount 的完整功能，但日常我更多使用 fava 这个图形界面。

在 fava 中已经预设了“资产负债表”，“损益表” 等常见财务用表，并且提供图表用于资产可视化，也具备多账本管理、预算管理等 beancount 自己没有的能力，更方便一些。

日常使用时，只需要在 beancount 账本文件中正常记账，然后调用 fava 运行即可。比如账本入口文件是 main.bean，那么在终端中运行：

```bash
$ fava main.bean
```

默认可以在浏览器 https://127.0.0.1:5000 访问 fava 主界面：

![]()

当然，实际应用中的账本比这复杂很多，Fava 提供了一个 Demo 站点，你可以在里面看看一个实际账本是如何运作的：[Fava Example](http://fava.pythonanywhere.com/example-with-budgets/balance_sheet/)。

### Fava 预设报表

在 Fava 右侧侧边栏，预设了常用报表，稍微解释一下：

- 损益表：呈现利润、收支、收支详情的报表。你可以在这里看到自己赚的、花的、从哪儿赚的钱，都花在哪儿了。
- 资产负债表：呈现净值、资产、负债、权益相关的信息。你可以在看到自己的资产净值、资产、负债和权益。
- 试算表：常用于对账。
- 日记账：以日期方式排序的流水账。
- 资产：查看自己的所有资产（Assets）账户
- 货币：查看所有“货币”对象的走势，例如人民币兑美元走势，需要使用 price 语法告诉 beancount 货币的价格。
- 事件：查看所有事件。例如一次旅行在账本里表现就是一个事件，用 event 语法定义
- 统计：整个账本的统计信息，例如账户内有多少笔转账之类。

在侧边栏下方，可以直接使用 Fava 编辑账本和新增交易，也可以从别的 beancount 账本中导入交易。

如果要设定 Fava 选项，例如中文界面，则需要在 main.bean 中加入以下语句

```
1990-01-01 custom "fava-option" "language" "zh"
```

然后重启 Fava 即可生效了。更多使用方法，可以点击侧边栏的 “帮助” 了解。

