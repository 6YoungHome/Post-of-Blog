---
title: 【编程学习】利用Python进行滚动线性回归
typora-root-url: 【编程学习】利用Python进行滚动线性回归
cover: /img/blog_img/15.png
description: 利用Python进行滚动线性回归，numpy_ext.rolling_apply方案以及pyfinance.ols.PandasRollingOLS方案
tags:
  - Python
  - 有用的代码
categories:
  - 编程学习
  - Python
abbrlink: ae058311
date: 2023-07-12 12:01:49
---







# 方案1：numpy_ext.rolling_apply

基于numpy_ext.rolling_apply的滚动回归需要自己定义一个回归计算的函数，可以直接使用sklearn或者statsmodels库中的方法进行回归计算。

## 相关环境准备

首先安装numpy_ext库，添加清华镜像源可以让下载更快。

```bash
# 以防有人没有，全部写出来吧
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple numpy_ext
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pandas
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple numpy
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple scikit-learn
```

导入相关环境

```python
import numpy as np
import pandas as pd
from numpy_ext import rolling_apply
from sklearn.linear_model import LinearRegression as LR
```

## 滚动回归实现

```python
def rolling_regression(s1,s2):
	'''线性回归函数，基于sklearn'''
    reg=LR().fit(pd.DataFrame(s1),pd.DataFrame(s2))
    return reg.coef_[0][0], reg.intercept_[0]

# 随机生成一个200*3的DataFrame
df = pd.DataFrame(np.random.random((200, 3), ), columns=['y', 'x1', 'x2'])

# rolling_apply(回归函数,滚动窗口,Y,X)
df[['beta', 'alpha']] = rolling_apply(rolling_regression, 20, df['y'].values, df['x1'].values)
```

## 方法优劣

**优势：**

- 方法简单
- 可以自定义相关函数

**劣势：**

- 用时相对较慢（比遍历快，比下一个方法慢）
- 只进行一元回归
- 所有需要的指标与结果都需要自定义进行计算



# 方案2：pyfinance.ols.PandasRollingOLS

首先安装pyfinance库，添加清华镜像源可以让下载更快。

```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyfinance
```

导入相关环境

```python
import numpy as np
import pandas as pd
from pyfinance.ols import RollingOLS, PandasRollingOLS
```

## 滚动回归实现

```python
# 随机生成一个200*3的DataFrame
df = pd.DataFrame(np.random.random((200, 3), ), columns=['y', 'x1', 'x2'])

# 直接进行滚动回归
# PandasRollingOLS(y, X, windows)
r = PandasRollingOLS(df.iloc[:, 0], df.iloc[:, 1:3], 20)

# 结果保存
df['alpha'] = t.alpha
df[['beta1', 'beta2']] = t.beta
```

## 其他计算结果

使用PandasRollingOLS方法不只能计算回归的结果，其他的相关指标也都一并计算了出来。以下是指标名称之间的对应关系：

|     方法名称     |    返回结果    |   方法名称   |                返回结果                 |  方法名称   |      返回结果      |
| :--------------: | :------------: | :----------: | :-------------------------------------: | :---------: | :----------------: |
|      alpha       | 常数项$\alpha$ |    ms_reg    |                 模型MSE                 |   ss_err    |     类内和方差     |
|       beta       | 斜率项$\beta$  |      n       |                窗口大小                 |   ss_reg    |     类间和方差     |
| condition_number |     条件数     |    names     |                变量名称                 |   ss_tot    |      总和方差      |
|      df_err      |   残差自由度   |  predicted   |            每次回归y的预测值            |   std_err   |     模型标准误     |
|      df_reg      |   模型自由度   | pvalue_alpha |                常数项p值                | tstat_alpha | 常数项的t统计量值  |
|      df_tot      |    总自由度    | pvalue_beta  |                 斜率p值                 | tstat_beta  |  自变量的t统计量   |
|  durbin_watson   |     DW检验     |    resids    |             每次回归的残差              |  use_const  |       不确定       |
|      fstat       |    F统计量     |     ridx     |          有回归结果的序列时间           |   window    |      窗口大小      |
|    fstat_sig     |   F统计量p值   |     rsq      |               拟合优度R^2               |      x      | 自变量(包含截距项) |
|    has_const     |     不确定     |   rsq_adj    |              调整拟合优度               |    xwins    |  每个窗口的自变量  |
|      index       |  全部序列时间  |   se_alpha   |             常数项标准误差              |      y      |       因变量       |
|   jarque_bera    |   J-B检验值    |   se_beta    |             斜率项标准误差              |    ybar     |   窗口因变量均值   |
|        k         |   自变量个数   |   solution   | 拟合出的所有参数<br> ($\alpha$在第一列) |    ywins    |  每个窗口的因变量  |
|      ms_err      |    残差MSE     |              |                                         |             |                    |



## 方法优劣

**优势：**

- 方法简单
- 可以进行多元线性回归
- 代码效率较高，计算快
- 相关指标容易获得

**劣势：**

- 不能自定义相关函数

















