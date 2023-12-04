---
title: 【量化金融】股票市场基础技术指标RSI
typora-root-url: 【量化金融】股票市场基础技术指标RSI
description: 简单介绍股票市场基础技术指标RSI，并分享python计算与绘图方法
cover: /img/blog_img/19.png
tags:
  - 量化金融
  - RSI
  - 技术指标
  - mplfinance
categories:
  - 量化金融
  - 基础知识
  - 技术指标
abbrlink: 24009e7e
date: 2023-10-18 16:10:29
---



## 什么是RSI

相对强弱指数(Relative Strength Index, RSI)股票市场是技术分析中常使用的动量指标。它能够衡量股票近期价格变化的速度和幅度，从而评估该股票价格是否被高估或低估。

RSI不仅可以发现超买和超卖的股票，还可以发现准备进行价格趋势逆转或纠正回调的股票；当然它也可以提示投资者在何时买入和卖出。一般来说，RSI读数达到70或以上表示超买情况，而读数达到30或以下表示超卖状态。

RSI比较了股票在价格上涨的日子和价格下跌的日子的强度。将这种比较的结果与价格走势相关联可以让交易者了解一种股票可能的表现。RSI与其他技术指标一起使用，可以帮助交易者做出更明智的交易决策。

## RSI的计算方法

$$
RS = \frac{ts\_mean(gain,14)}{ts\_mean(loss,14)} \\
RSI = 100-\frac{100}{1+RS}
$$



## Python计算与绘图

为了绘制一只股票的RSI线，我们可以从Tushare方便的下载股票高开低收量等数据。

```python
import tushare as ts

# 获取股票数据
api = ts.pro_api("你的token")
df = api.daily(ts_code="000001.SZ", start_date=f'20221001', end_date=f'20231231')

# 股票数据初步处理
df = df.rename(columns={'ts_code': 'stock_code', 'trade_date': 'date', 'vol': 'volume'})
df['date'] = pd.to_datetime(df['date'])
df = df.set_index(df['date']).sort_index()
```

然后计算股票RSI指标。

```python
import pandas as pd

rsi = pd.DataFrame()
rsi['gain'] = df['change'].apply(lambda x: x if x > 0 else 0)
rsi['loss'] = df['change'].apply(lambda x: x if x < 0 else 0)
rsi['avg_gain'] = rsi['gain'].rolling(14).mean()
rsi['avg_loss'] = rsi['loss'].rolling(14).mean()
rsi['rs'] = rsi['avg_gain'] / rsi['avg_loss']
rsi['rsi'] = 100 - 100 / (1 - rsi['rs'])
rsi['up'] = 70
rsi['down'] = 30
```

使用mplfinance库，绘制K线图以及RSI等相关指标。

```python
from matplotlib import pyplot as plt
import mplfinance as mpf

plt.rcParams['font.sans-serif'] = ['SimHei'] # 显示中文
plt.rcParams['axes.unicode_minus'] = False # 显示负号

mc = mpf.make_marketcolors(up='r', down='g', edge='inherit', wick='inherit', volume='inherit')
style = mpf.make_mpf_style(base_mpl_style = 'default', marketcolors=my_color, figcolor='(0.82, 0.83, 0.85)', gridcolor='(0.82, 0.83, 0.85)')


add_plot = [
    mpf.make_addplot(rsi.rsi.loc[str(year)], panel='lower', label="RSI"),
    mpf.make_addplot(rsi.up.loc[str(year)], linestyle="--", panel='lower', label="UP"),
    mpf.make_addplot(rsi.down.loc[str(year)], linestyle="--", panel='lower', label="DOWN"),
]

fig, ax = mpf.plot(df.loc[str(year)],
    type = "candle", style=style, addplot = add_plot,
    title = f"Market of stock {stock_code} in {year} year",
    ylabel = "price", ylabel_lower = 'RSI',
    figratio = (16, 9), returnfig=True
)
```

![261ab945-9dff-4b24-9a9a-f7011cbbf42c](/261ab945-9dff-4b24-9a9a-f7011cbbf42c.png)

## 将 RSI 与趋势结合使用

RSI指标在应用时要注意，它在波段区间中表现最佳，而不是在趋势市场中。如何调整从而让其更加适合趋势市场呢？

在长期的下降趋势中，RSI的峰值会在50左右，而不是在70（可以简单参考上图2-7月的表现），而此时，50水平线会被认为是更加可靠的看跌信号

当强劲趋势到位时，研究者会在 30 和 70 的水平之间创建一条水平趋势线，以更好地识别整体趋势和极端情况。而当股票或资产的价格处于长期水平通道或交易区间（而不是强劲的上升或下降趋势）时，通常不需要修改超买或超卖 RSI 水平，直接使用30/70即可。

## 如何理解超买或超卖

通常，当 RSI 指标在 RSI 图表上穿过 30 时，它是一个看涨信号，当它穿过 70 时，它是一个看跌信号。换句话说，人们可以解释为RSI值为70或更高表明股票正在超买或被高估。它可能为趋势逆转或修正性价格回调做好了准备。RSI 读数为 30 或更低表示超卖或低估状况。

超买，顾名思义，买的太多，超过了，它是指以高于其真实（或内在）价值的价格水平进行交易的股票。这意味着它的价格高于应有的水平，根据技术分析或基本面分析的从业者。看到股票超买迹象的交易者可能会预期价格修正或趋势逆转，因此会出售股票。反之亦然。

## RSI 背离

当价格向与 RSI 相反的方向移动时，就会发生 RSI 背离。换句话说，图表可能会在价格的相应变化之前显示动量的变化。

当 RSI 显示超卖读数后出现较高的低点时，就会出现看涨背离，而价格的低点较低。这可能表明看涨势头上升，突破超卖区域可能被用来触发新的多头头寸。

当 RSI 产生超买读数后出现较低的高点时，就会出现看跌背离，该高点出现在价格的高点上。

如下图所示，当 RSI 形成较高的低点时，当价格形成较低的低点时，确定了看涨背离。这是一个有效的信号，但当股票处于稳定的长期趋势时，背离可能很少见。使用灵活的超卖或超买读数将有助于识别更多潜在信号。

![image-20231018154513150](/image-20231018154513150.png)

## 正负 RSI 反转示例

交易者寻找的另一个价格-RSI 关系是正 RSI 反转和负数。一旦 RSI 达到低于其先前低点的低点，同时证券价格达到高于其先前低点的低点，则可能发生正 RSI 反转。交易者会认为这种形态是看涨信号和买入信号。

相反，一旦 RSI 达到高于其先前高点的高点，同时证券价格达到较低的高点，则可能会发生负 RSI 反转。这种形态将是一个看跌信号和卖出信号。

这一分析思路与上一小节RSI背离的特征很像，但是是相反的，要注意记忆和辨别。

## RSI 摆幅抑制示例

另一种交易技术检查 RSI 从超买或超卖区域重新出现时的行为。该信号称为看涨摆动拒绝，由四个部分组成：

1. RSI 陷入超卖区域。
2. 相对强弱指标回升至30以上。
3. RSI形成另一个下跌，没有回到超卖区域。
4. RSI随后突破近期高点。

如下图所示，RSI 指标超卖，突破 30，并形成拒绝低点，当信号反弹走高时触发信号。以这种方式使用 RSI 与在价格图表上绘制趋势线非常相似。

## RSI 的局限性

RSI 比较看涨和看跌的价格动量，并在价格图表下方的振荡器中显示结果。与大多数技术指标一样，当其信号符合长期趋势时，它们最可靠。

真正的反转信号很少见，很难与误报区分开来。例如，误报将是看涨交叉，然后是股票突然下跌。假阴性是指出现看跌交叉的情况，但股票突然加速上涨。

由于该指标显示动量，因此当资产在任一方向上具有显着动量时，它可以长时间保持超买或超卖。因此，RSI 在资产价格在看涨和看跌走势之间交替的振荡市场（交易范围）中最有用。



