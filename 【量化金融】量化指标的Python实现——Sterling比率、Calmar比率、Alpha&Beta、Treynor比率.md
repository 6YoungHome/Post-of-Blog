---
title: 【量化金融】量化指标的Python实现——Sterling比率、Calmar比率、Alpha&Beta、Treynor比率
typora-root-url: 【量化金融】量化指标的Python实现——Calmar比率、Sterling比率、Alpha&Beta、Treynor比率
description: 利用Python实现量化策略评价指标：Calmar比率、Sterling比率、Alpha&Beta、Treynor比率
cover: /img/blog_img/19.png
tags:
  - 量化金融
  - Calmar比率
  - Sterling比率
  - Alpha&Beta
  - Treynor比率
  - 量化评价指标
  - Python
categories:
  - 量化金融
  - 代码实现
  - 量化评价指标
abbrlink: eb7238e
date: 2023-07-31 12:43:53
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



# Calmar比率

Calmar比率描述的是收益和最大回撤之间的关系。计算方式为年化收益率与历史最大回撤之间的比值。Calmar比率数值越大，策略的业绩表现越好。反之，策略的表现越差。

## Python实现

```python
def calmar_ratio(returns_df):
    annual_ret = annual_return(returns_df, period)['annual_return']
    mdd = maximum_drawdown(returns_df)['max_drawdown']
    cr = annual_ret / mdd
    res_dict = {'calmar_ratio': cr}
    return res_dict
```

其中annual_return与maximum_drawdown函数见文章：

<a href="https://www.6young.site/blog/f73d6124.html" target="cardlink_"></a>



# Sterling比率

Sterling比率是衡量投资组合风险调整后回报的指标，这个指标有很多定义方式，但是总体的思路与特征是一致的，在这里我选取一种个人感觉最好的计算方式进行分享。
$$
sterling\_ratio=\frac{r_p-r_f}{average\_maxdrawdown}
$$
将之与Calmar比率进行对比，sterling比率使用近三年最大回撤的均值，更具有普遍性，对异常值更加鲁棒。

## Python实现方式

```python
def sterling_ratio(returns_df, period='yearly'):
    period_days = get_period_days(period)
    s = 0
    for i in range(3):
        returns_df_y = returns_df.iloc[-1-(i+1)*period_days:-1-i*period_days]
        s += maximum_drawdown(returns_df_y)[0]
    s = s / 3 + 0.1
    slr = annual_return(returns_df, period=period) / s
    res_dict = {'sterling_ratio': slr}
    return res_dict
```



# Alpha&Beta

这是学习金融课程是最先接触的几个概念之一，确实不应该放在这么后面来分享。

其中，Alpha衡量策略相对于市场的超额收益，即策略的独立收益部分。Beta衡量策略相对于市场指数的敏感性，Beta值大于1表示策略波动性大于市场，小于1表示波动性小于市场。

## Python数学方法实现

```python
def alpha_beta(ret_i, ret_m, risk_free=0, period='yearly'):
    period_days = get_period_days(period)
    beta = ret_i.cov(ret_m)/ret_m.var()
    alpha = ((ret_i.mean()-risk_free-beta*(ret_m-risk_free).mean())+1)**period_days-1
    res_dict = {'alpha': alpha, 'Beta': beta}
    return res_dict
```

## Python回归方法实现

```python
from sklearn.linear_model import LinearRegression
def alpha_beta(ret_i, ret_m, risk_free=0, period='yearly'):
    period_days = get_period_days(period)
    LR = LinearRegression()
    LR.fit(pd.DataFrame(ret_m-risk_free), pd.DataFrame(ret_i-risk_free))
    beta = LR.coef_[0][0]
    alpha = (1+LR.intercept_[0])**period_days-1
    res_dict = {'alpha': alpha, 'Beta': beta}
    return res_dict
```



# Treynor比率

Treynor比率（特雷诺比率），也称为回报与波动率比率，和夏普比率类似，用于描述投资组合承担的每个风险单位产生了多少超额回报，只是特雷诺比率中的风险是指以投资组合的贝塔系数衡量的系统性风险，而不是组合收益的方差。

较高的Treynor比率结果意味着投资组合是更合适的投资。
$$
treynor\_ratio=\frac{r_p-r_f}{\beta_p}
$$

## Python实现

```python
def treynor_ratio(returns_df, ret_m, risk_free=0, period='yearly'):
    period_days = get_period_days(period)
    rp_f = (returns_df.mean() - risk_free)
    beta = alpha_beta(returns_df, ret_m, risk_free, period)['beta']
    tr = rp_f * (period_days ** 0.5) / beta
    res_dict = {'treynor_ratio': tr}
    return res_dict
```

