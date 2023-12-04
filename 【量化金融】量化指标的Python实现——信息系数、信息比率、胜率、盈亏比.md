---
title: 【量化金融】量化指标的Python实现——信息系数、信息比率、胜率、盈亏比
typora-root-url: 【量化金融】量化指标的Python实现——信息系数、信息比率、胜率、盈亏比
cover: /img/blog_img/1.png
description: 利用Python实现量化策略评价指标：信息系数、信息比率、胜率、盈亏比
tags:
  - 量化金融
  - 信息比率IR
  - 信息系数IC
  - 胜率
  - 盈亏比
  - 量化评价指标
  - Python
categories:
  - 量化金融
  - 代码实现
  - 量化评价指标
abbrlink: 55da4fa7
date: 2023-08-02 09:26:41
---



{% link Quantools项目地址,6YoungHome,https://github.com/6YoungHome/Quantools %}



# 环境准备

测试用数据：[demo_close_price](/download/close_price_demo.csv)点击下载

测试用数据：[factor_and_return](https://pan.baidu.com/s/1tAKpzKoRXBfHE2mIhldaBA?pwd=0ovc )点击下载

```python
import pandas as pd
import numpy as np

cp = pd.read_csv('close_price_demo.csv', encoding='gbk', index_col='date')
cp.index = pd.to_datetime(cp.index)
ret = cp.pct_change().iloc[1::]
cum_ret = (1+ret).cumprod()
cum_ret.plot(figsize=(10, 4))
```



```python
# 计算年(季/月/周)化收益的相关常数
BDAYS_PER_YEAR = 252
BDAYS_PER_QTRS = 63
BDAYS_PER_MONTH = 21
BDAYS_PER_WEEK = 5

DAYS_PER_YEAR = 365
DAYS_PER_QTRS = 90
DAYS_PER_MONTH = 30
DAYS_PER_WEEK = 7

MONTHS_PER_YEAR = 12
WEEKS_PER_YEAR = 52
QTRS_PER_YEAR = 4

def get_period_days(period):
    period_days = {
        'yearly': BDAYS_PER_YEAR, 'quarterly': BDAYS_PER_QTRS,
        'monthly': BDAYS_PER_MONTH, 'weekly': BDAYS_PER_WEEK, 'daily': 1, 
        
        'monthly2yearly': MONTHS_PER_YEAR,
        'quarterly2yearly': QTRS_PER_YEAR, 
        'weekly2yearly': WEEKS_PER_YEAR, 
    }
    return period_days[period]
```





# 信息系数（Information Coefficient）

信息系数（Information Coefficient，IC）是一种常用的评估投资策略预测能力的指标。IC衡量投资策略的预测结果与实际收益之间的相关性，即策略预测的准确性。IC的取值范围在-1到1之间，数值越接近1表示预测能力越强，数值越接近-1表示预测能力越弱，接近0表示预测能力不显著。信息系数也分为两类：

1. Normal IC 皮尔逊相关关系

$$
Normal IC = corr(f_{t-1}-r_t) \\

f_{t-1}：t-1期股票因子值 \\

r_t：t期股票收益率
$$

2. Rank IC 斯皮尔曼相关系数

$$
Rank IC = corr(order_{t-1}^f-order_t^r) \\

order_{t-1}^f：t-1期股票因子值排名 \\

order_t^r：t期股票收益率排名
$$

当IC>0.05，就可以视为有效因子；当IC>0.1，就可以认为是很好的阿尔法因子了；当IC均值接近0，可视为无效因子。

## Python实现

```python
def cal_ic(factor_b, return_n, method="normal"):
    """_summary_

    Args:
        factor_b (_pd.DataFrame_): t-1期因子数据
        return_n (_pd.DataFrame_): t期收益率数据（与factor相互对应）
        method (_str_, optional): 默认为"normal", 还可以是"rank".
    
    returns:
        ic: 信息系数
    """
    factor_b = factor_b.reset_index(drop=True)
    return_n = return_n.reset_index(drop=True)

    df = pd.concat([factor_b, return_n], axis=1)
    if method == "rank":
        df = df.rank()
    ic = df.corr().iloc[0,1]
	res_dict = {"IC": ic}
    return res_dict
```



# 信息比率（Information Ratio）

信息比率事实上分为两种，一种是描述策略稳定战胜基准指数的能力的“策略信息比率”，另一种是评价因子选股能力的“IC信息比率”。

## 策略信息比率

策略信息比率（Information Ratio，IR）是一种用于衡量投资策略的超额收益相对于基准指数的风险的指标。它是一种风险调整后的绩效度量，可以帮助投资者判断策略的风险调整后的表现，并评估策略的主动管理能力。策略信息比率基于以下两个关键概念：

1. **超额收益（Excess Return）**：超额收益是策略相对于其参考基准的收益。它表示策略的绝对表现，即策略是否能够产生高于基准的收益。
2. **跟踪误差（Tracking Error）**：跟踪误差是策略收益与基准收益之间的标准差，它衡量了策略相对于基准的波动性。较高的跟踪误差表示策略与基准的差异较大。

策略信息比率的计算公式如下：
$$
strategy\_IR=\frac{\bar{r_p}-\bar{r_{index}}}{\sigma_{(r_p-r_{index})}}
$$

### Python实现

```python
def strategy_ir(return_p, return_i):
    excess_return = return_p-return_i
    ir = excess_return.mean() / excess_return.std()
    res_dict = {"strategy_infomation_ratio": ir}
    return res_dict
```

## IC信息比率



IC信息比率（Information Coefficient Information Ratio，ICIR）是一种衡量投资策略预测能力的指标。与传统的策略信息比率类似，IC信息比率也是一个风险调整后的绩效度量，用于评估策略中的信号或预测与实际收益之间的关联性。IC信息比率结合了两个重要概念：

ICIR可以通过以下公式进行计算：
$$
IC\_IR=\frac{\bar{ic}}{\sigma_{ic}}
$$
ICIR兼顾了因子的选股能力（由IC表征）和选股能力的稳定性（由IC的标准差的倒数代表），因此他可以较好地帮助投资者评估策略预测能力的质量。

### Python实现

```python
def ic_ir(factor_df, return_df, method='normal'):
    """_summary_

    Args:
        factor_df (_pd.DataFrame_): columns: dates; index: tickers; values: factors
        return_df (_pd.DataFrame_): columns: dates; index: tickers; values: returns
        method (_str_, optional): 默认为"normal", 还可以是"rank".
    return:
        icir: ICIR
    """
    factors_b = factor_df.shift(1, axis=1)
    ic_ls = []
    for i in range(1, factors.shape[1]):
        date = factors.columns[i]
        ic = cal_ic(factors_b[date], return_df[date], method=method)["IC"]
        ic_ls.append(ic)
    ic_ir = np.mean(ic_ls) / np.std(ic_ls)
    res_dict = {"ICIR": ic_ir}
    return res_dict
```



# 胜率

胜率（Win Rate），也称为获胜率或胜利率，是一个用于衡量投资或交易策略中获胜交易占总交易数量的比例的指标。

胜率是一个很直观的指标，它反映了交易策略在多少交易中能够取得盈利。高胜率通常被视为一个积极的特征，因为它意味着策略在大部分交易中都能取得盈利。

## Python实现

```python
def win_rate(return_df):
    win_trades = np.sum(return_df > 0)
    total_trades = len(return_df)
    res_dict = {"win_rate": win_trades / total_trades}
    return res_dict
```



# 盈亏比

盈亏比（profit-loss ratio）是用于衡量投资或交易策略盈利能力的一个指标，它反映了盈利交易额与亏损交易额之间的比率。盈亏比可以帮助投资者判断策略的盈利性和风险管理能力。

## Python实现

```python
def profit_loss_ratio(return_df):
    total_profit = np.sum(return_df[return_df > 0])
    total_loss = abs(np.sum(return_df[return_df < 0]))
    res_dict = {"profit_loss_ratio": total_profit / total_loss}
    return res_dict
```

