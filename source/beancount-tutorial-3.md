---
title: 3. 导入浦发银行信用卡账单
brief: 使用 Python 脚本导入浦发银行信用卡账单
date: 2019-04-25
tags: ["Beancount", "复式记账", "Beancount 教程"]
---

在[上一篇](beancount-tutorial-2)中，我介绍了使用 Beancount 命令和 Fava 来查账。本篇会介绍如何使用 Python 脚本和信用卡账单，自动生成 Beancount 账本。

---

## 为什么使用信用卡

信用卡是一种简单借贷服务：使用信用卡的每次消费都是在向银行借钱，账单期内还清账单。

这样的消费方式有很多好处。比如说让现金流更加充裕，获得更高的银行信用等等。不过对我们记账工作来说，使用信用卡有一项很便利的优点：**它定期提供每个月消费账单供我们查阅**。

如果日常购物消费都使用信用卡，那么就省去了手工记账的麻烦了；而且这种做法也便于管理负债。

一次普通的消费，在 Beancount 中一般记录成：

```beancount
2019-04-05 * "支付宝" "上海喜士多便利连锁有限…, 零食"
    Liabilities:SPDB                    -5.4 CNY    ; SPDB 是浦发银行简称
    Expenses:Food:Snack                 +5.4 CNY
```

其中，`SPDB` 是浦发银行的简称。

可以看到，使用信用卡购买 ¥5.4 零食，是从 `Liabilities` (负债)中减去 5.4，然后加到了 `Expenses:Food:Snack` (零食) 中。

如果是还款，则这样记录：

```beancount
2019-04-08 * "银行还款"
    Assets:Cash                     -8289.89 CNY
    Liabilities:SPDB                +8289.89 CNY
```

使用现金还清了 8289.89 元欠款。


## 导出浦发银行信用卡账单

很多信用卡网银无法导出原始版本的信用卡账单，不过浦东发展银行可以导出 `.xls` 格式的账单。所以我们以浦发银行为例。

浦发银行在官网的[这个入口](https://ebill.spdbccc.com.cn/cloudbank-portal/loginController/toLogin.action) 提供了账单查询入口。选择账单期后，点击查询，在右上角的“账单明细”中，可以选择“下载账单”

下载完成后 xls 版本的账单，我用 [Numbers](https://www.apple.com/numbers/) 手工把它转换成 csv 版本。当然，你也可以用 Excel 或者写一个脚本来将这个过程完全自动化。

得到的 csv 文件如果用文本编辑器打开大概是这个样子：

```csv
交易日期,记账日期,交易描述,卡号末四位,卡片类型,交易币种,交易金额
20190408,20190408,支付宝 上海喜士多便利连锁有限…,1234,主卡,人民币,5.40
20190403,20190403,支付宝 上海拉扎斯信息科技有限…,1234,主卡,人民币,142.30
……
```

当月所有的消费账单都记录在此，每行一条。接下来我们需要解析其中的内容，然后转换为 Beancount 格式记录。

## 生成 Beancount 记录

为了让这个过程省心，我写了一个脚本 [beancount-converter](https://github.com/lyricat/beancount-converter)。

运行脚本，输入账单 csv 文件，可以输出对应的 Beancount 文件：

```bash
$ ./proc.py -m spdb -f historical/2018-08-spdb.csv
```

这里需要注意，使用之前，需要创建配置两个文件：`config.json` 和 `mapping.json`，你可以用 repo 中的 example 来创建：

```bash
$ mv mapping.example.json mapping.json
$ mv config.example.json config.json
```

并且编辑他们的内容，把 config.json 里的账户名称换成你的账户，然后在 mapping.json 里补充交易的关键字。mapping.json 的格式是这样的：

```javascript,ignore
[
//  交易详情中的关键字, 在 Beancount 中的描述，在 Beancount 中的分类
	["肯德基", 		"吃饭",  "Expenses:Food:General"],
	["必胜客", 	    "吃饭",  "Expenses:Food:General"], 
]
```

比如一条信用卡交易是在肯德基买了吃的：

```csv
...
20190328,20190328,财付通 肯德基,1234,主卡,人民币,93.80
...
```

那么，根据 mapping.json 中的内容，会得到如下 Beancount 记录：

```beancount
2019-03-28 * "财付通" "肯德基, 吃饭"
    Liabilities:SPDB                    -93.8 CNY
    Expenses:Food:General               +93.8 CNY
```

根据自己的需要，自行补充 mapping.json 中的内容，可以实现大部分的交易的自动化记录。无法识别的交易会记录在 `Expenses:Unknown` 下，需人工确认。

不过这也很快，基本上十分钟内就搞定。




