---
title: Beancount 速查表
date: 2019-03-05
tags: ["Beancount", "复式记账", "Beancount 教程"]
---

最初版本来自 [Beancount Syntax Cheat Sheet](https://docs.google.com/document/d/1M4GwF6BkcXyVVvj4yXBJMX7YFXpxlxo95W6CpU3uWVc/edit#)

## 基础

账户名称示例：`Assets:US:BofA:Checking`

账户类型：

```
Assets          +
Liabilities     -
Income          -
Expenses        +
Equity          -
```


货币（全大写）：

```
USD, EUR, CAD, AUD
GOOG, AAPL, RBF1005
HOME_MAYST, AIRMILES
HOURS
```

## 命令

基本语法：

```beancount
YYYY-MM-DD <directive> <arguments...>
```

创建、关闭账户：

```beancount
2001-05-29 open Expenses:Restaurant
2001-05-29 open Assets:Checking     USD,EUR  ; Currency constraints

2015-04-23 close Assets:Checking
```

声明货币，添加元信息（可选）：

```beancount
1998-07-22 commodity AAPL
  name: "Apple Computer Inc."
```

记录价格：

```beancount
2015-04-30 price AAPL   125.15 USD
2015-05-30 price AAPL   130.28 USD
```

备注：

```beancount
2013-03-20 note Assets:Checking "Called to ask about rebate" 
```

附加文档：

```beancount
2013-03-20 document Assets:Checking "path/to/statement.pdf"
```

交易：

```beancount
2015-05-30 * "Some narration about this transaction"
  Liabilities:CreditCard   -101.23 USD
  Expenses:Restaurant       101.23 USD


2015-05-30 ! "Cable Co" "Phone Bill" #tag ˆlink
  id: "TW378743437"               ; Meta-data
  Expenses:Home:Phone  87.45 USD
  Assets:Checking                 ; You may leave one amount out
```

记账：

```beancount
  ...    123.45 USD                             Simple
  ...        10 GOOG {502.12 USD}               With per-unit cost
  ...        10 GOOG {{5021.20 USD}}            With total cost
  ...        10 GOOG {502.12 # 9.95 USD}        With both costs
  ...   1000.00 USD   @ 1.10 CAD                With per-unit price
  ...        10 GOOG {502.12 USD} @ 1.10 CAD    With cost & price
  ...        10 GOOG {502.12 USD, 2014-05-12}   With date 
  ! ...   123.45 USD ...                        With flag
```

账户断言和补垫：

```beancount
; 断言制定货币的金额为 -634.30 USD
2015-06-01 balance Liabilities:CreditCard  -634.30 USD 

; 自动插入交易以完成以下断言
2015-06-01pad Assets:Checking Equity:Opening-Balances
```

事件：

```beancount
2015-06-01 event "location" "New York, USA"
2015-06-30 event "address" "123 May Street"
```

选项：

```beancount
option "title" "My Personal Ledger"
```

有关支持的选项的完整列表，请参阅[此文档](http://furius.ca/beancount/doc/options)。

其它：

```beancount
pushtag #trip-to-peru
...
poptag  #trip-to-peru
; Comments begin with a semi-colon
```

