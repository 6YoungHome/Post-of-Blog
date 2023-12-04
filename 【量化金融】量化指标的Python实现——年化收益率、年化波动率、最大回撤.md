---
title: 【量化金融】量化指标的Python实现——年化收益率、年化波动率、最大回撤
typora-root-url: 【量化金融】量化指标的Python实现——年化收益率、年化波动率、最大回撤
cover: /img/blog_img/17.png
description: 利用Python实现量化策略评价指标：年化收益率、年化波动率、最大回撤
tags:
  - 量化金融
  - 年化收益率
  - 年化波动率
  - 最大回撤
  - 量化评价金融
  - Python
categories:
  - 量化金融
  - 代码实现
  - 量化评价指标
abbrlink: f73d6124
date: 2023-07-29 11:43:53
---



{% link Quantools项目地址,6YoungHome,https://github.com/6YoungHome/Quantools %}



# 环境准备

测试用数据：[demo_close_price_data](/download/close_price_demo.csv)点击下载

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



# 年化收益率

年化收益率是指你的投资策略在一年（252个交易日）所能获得的平均收益情况，是最直观反映策略优劣的指标之一。

假设在你的t日回测期间，你的本金从$p_0$变为了$p_t$，那么在这一时间段中，你所获得收益的年化收益率$r_y$可以通过以下方式计算：
$$
r_y=(\frac{p_t-p_0}{p_0}+1)^{\frac{252}{t}}-1
$$
其中，$r_b=\frac{p_t-p_0}{p_0}$为你在回测期间的总收益率。

## Python实现方式

```python
def annual_return(returns_df, period='yearly'):
    period_days = get_period_days(period)
    total_return = (returns_df + 1).prod(axis=0)
    annual_ret = total_return ** (period_days / returns_df.shape[0]) - 1
    res_dict = {'annual_return': annual_ret}
    return res_dict
```



# 年化波动率

年化波动率是指你的投资策略在一年（252个交易日）所能获得的平均收益的波动情况，能最基础的地反映策略的风险特征。

同样是在t日回测期间。我们首先计算出以日为频率的方差：$\sigma_d=\sqrt{\frac{\sum(r_i-\bar r)^2}{t}}$

我们假设每日的收益率之间是不相关的（这个假设显然太强了，不符合现实），因此根据方差具有可加性的原理，一年（252交易日）的波动率就为：$\sigma_y=\sqrt{252}\sigma_d$

## Python实现

```python
def annual_volatility(returns_df, period='yearly'):
    period_days = get_period_days(period)
    annual_vol = returns_df.std() * (period_days ** 0.5)
    res_dict = {'annual_volatility': annual_vol}
    return res_dict
```



# 最大回撤

最大回撤是描述你的投资策略在回测期间所遭受的最大损失的大小，反映了一个策略的风险承受能力。

我们要注意，最大回撤区间的开始点不一定是最高点，结束点也不一定是最低点，因此不能想当然的去用最高减去最低，要根据实际情况进行讨论。

## Python实现

```python
def maximum_drawdown(returns_df):
    """_summary_

    Args:
        returns_df (_pd.DataFrame_): 收益率的DataFrame

    Returns:
        mdd: 最大回撤
        start: 回撤期开始时间点
        end: 回撤期结束时间点
    """
    cum_returns = (returns_df + 1).cumprod(axis=0)
    peak = cum_returns.expanding().max()
    dd = ((peak - cum_returns)/peak)
    mdd = dd.max()
    end = dd[dd==mdd].dropna().index[0]
    start = peak[peak==peak.loc[end]].index[0]
    res_dict = {
        'max_drawdown': mdd, 'max_drawdown_start': start,
        'max_drawdown_end': end, 
    }
    return res_dict
```













 
