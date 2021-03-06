---
title: 个人所得税
date: 2020-04-21 17:53:19
tags: 个人所得税
categories: [知识区]
---

2019 年 1 月 1 日 进行了税法改革，采用年度计算

#### 税额计算

应纳税所得额 = 综合所得额 - 免征 - 扣除
综合所得额 = 工资 + 劳务报酬 \* 80% + 稿酬 \* 80% \* 70% + 特许使用费

- 免征: 6w / 年
- 劳务报酬: 倘若一个程序员给别的公司开发软件，这部分的报酬即是劳务报酬
- 稿酬: 写作投稿的费用，可能是国家鼓励写作创作
- 特许使用费: 倘若考了证，给别的公司挂牌使用而得到的费用

<!-- more -->

- 扣除:
  1. 专项: 三险一金，养老保险 + 医疗保险 + 失业保险 + 住房公积金
  2. 附加:
  - 子女教育，1.2w / 年 / 个
  - 继续教育，如考上研究生，博士（4800 / 年），如果是考上教师资格证、会计证之类的（3600 / 年）
  - 大病医疗: 自付医疗费用 > 1.5w, 则扣除 1.5w，8w 封顶
  - 房贷:
    1. 首次房贷 1.2w / 年，第二套房子不免除
    2. 租房也可以享受，但是房贷和租房必须 2 选 1，大中小城市分别是 1.8w / 年，1.32w / 年，9600 / 年
  - 赡养老人: 独生子女 2.4w / 年， 非独生子女 1.2w / 年

#### 速算扣除数一览

![速算扣除数](https://cdn.jsdelivr.net/gh/aslkami/hexo-blog/source/images/tax.png)

倘若一个人的应纳税所得额 是 10w 元，则计算方法是 (10w \* 10 % -2520) 元

#### 劳务报酬扣除数一览

y 代表劳务应纳税收入
x 代表劳务收入

- x < 800, 则 y = x - 0
- x > 800 且 x < 4000, 则 y = x - 800
- x > 4000, 则 y = x \* 80%

![劳务应纳税收入速算扣除数](https://cdn.jsdelivr.net/gh/aslkami/hexo-blog/source/images/labour_services.png)

倘若一个人没有签劳动合同，而得到了 18w 的劳动报酬，则按上面算法

> 应缴纳税额 = 18w \* 80% \* 40% - 7000 = 5.06w
> 税后收入 = 18w - 5.06w = 12.94w

#### 年度扣除和阅读扣除比较

月度扣除不太合理

- 假设生了场大病，扣除了 8w 元，但是自己的收入都达不到 8w，政府还得给你钱
- 假设一个人前半年收入很高缴了很多税，后半年失业了，没收入，这样也不合理

#### 汇算清缴

倘若一个人分别在 A，B 公司工作半年， 每个月 1.5w
则预算扣除总额为(9w 属于 10%)

- A 公司：（1.5w \* 6 - 5000 \* 6）\* 10% - 2520 = 3480 元
- B 公司：3480 元
- 合计 6960 元

年度汇算的话

> (15w \* 12 - 6w ) \* 10% - 2520 = 9480 元
> 需要多缴纳 9480 - 6960 = 2520 元，需要在 APP 申报

如果是劳务的话，上面的例子是要缴纳 5.06w

> (18w \* 80% - 6w) \* 10% - 2520 = 5880 元
> 则在 app 申报可以退的金额 = 5.06w - 0.588w = 4.472 w

#### 以下情况可以不用申报

- 年收入 <= 12w
- 需要补税 <= 400 元
- 预缴和实际一致
- 放弃退税(国家给你钱你不要)

> 年终奖的话要么进行单列，要么放在综合所得额那里，哪种税低申报哪种
