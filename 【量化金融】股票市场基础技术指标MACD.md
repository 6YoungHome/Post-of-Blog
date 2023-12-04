---
title: 【量化金融】股票市场基础技术指标MACD
typora-root-url: 【量化金融】股票市场基础技术指标MACD
description: 简单介绍股票市场基础技术指标MACD，并分享python计算与绘图方法
cover: /img/blog_img/18.png
tags:
  - 量化金融
  - MACD
  - 技术指标
  - mplfinance
categories:
  - 量化金融
  - 基础知识
  - 技术指标
abbrlink: f91deefc
date: 2023-10-18 14:34:29
---

## 简介与计算方式

移动平均收敛/发散线（Moving Average Convergence/Divergence，MACD）是一种趋势跟踪的动量指标，它通过股票价格的两个指数移动平均线计算得出，一般来说，MACD线是通过股票收盘价12日移动平均值减去26日移动平均值。

此外，通过计算MACD的9日移动平均值，将之定义为“信号线”。将MACD线与信号线绘制于同一张图重，可以发现有效的买入、卖出信号：**当MACD线穿过信号线上方时买入；当MACD线穿过信号线下方时卖出。**该指标可以用多种方式解释，最常见的方法是研究MACD指标的交叉、背离以及快速上升和下降。
$$
MACD = SMA(close, 12) - SMA(close, 26) \\
Signal = SMA(MACD, 9)
$$



## 使用注意

用MACD指标分析市场时，有几个关键要点一定要注意：

1. MACD指标的计算方式：一般来说，MACD线是通过股票收盘价12日移动平均值减去26日移动平均值。信号线是MACD线的9日移动平均。但是这是美国人在美股市场中发现的规律，在中国用什么样的参数合适，在其他金融市场应该用什么样的参数仍然需要尝试。
2. MACD可以帮助衡量证券是超买还是超卖，提醒交易者方向性走势的强度，并警告潜在的价格反转。还可以提醒投资者看涨/看跌背离（例如，当价格新高未被MACD的新高确认时，表明潜在的失败和逆转，反之亦然）。
3. 信号线交叉后，建议等待三四天，以确认这不是假交叉。
4. **MACD是一个滞后指标。**毕竟，MACD中使用的所有数据都是基于股票的历史价格行为。因为它是基于历史数据的，所以它必然滞后于价格。但是，一些交易者使用MACD直方图来预测趋势何时会发生变化。对于这些交易者来说，MACD的这一方面可能被视为未来趋势变化的领先指标。



## Python绘图

MACD指标通常以直方图显示，直方图绘制MACD与其信号线之间的距离。如果MACD高于信号线，直方图将高于MACD的基线或零线。如果 MACD 低于其信号线，直方图将低于MACD的基线。交易者使用MACD的直方图来确定看涨或看跌，并发现其中的超买/超卖信号。

为了绘制一只股票的MACD线，我们可以从Tushare方便的下载股票高开低收量等数据。

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

然后计算MACD相关的众多指标。

```python
import pandas as pd

sma_26 = df.close.rolling(26).mean() # 26日均线
sma_12 = df.close.rolling(12).mean() # 12日均线
macd = sma_12 - sma_26 # MACD线
signal = macd.rolling(9).mean() # 信号线

# 计算MACD与信号线之差，为了绘制不同颜色，分开统计
histogram = macd - signal  
histogram[histogram < 0] = None  
histogram_positive = histogram
histogram = macd - signal  
histogram[histogram >= 0] = None  
histogram_negative = histogram
```

使用mplfinance库，绘制K线图以及MACD等相关指标。

```python
from matplotlib import pyplot as plt
import mplfinance as mpf

plt.rcParams['font.sans-serif'] = ['SimHei'] # 显示中文
plt.rcParams['axes.unicode_minus'] = False # 显示负号

mc = mpf.make_marketcolors(up = 'red', down = 'green', ohlc = "inherit", volume = "inherit", alpha = 0.6)
style = mpf.make_mpf_style(base_mpl_style = 'default', marketcolors = mc)

add_plot = [
    # 默认在第一个子图中添加
    mpf.make_addplot(sma_12.loc['2023'], label="SMA12"), 
    mpf.make_addplot(sma_26.loc['2023'], label="SMA26"),
    # panel=2绘制在第三个子图中
    mpf.make_addplot(macd.loc['2023'], panel=2, color='skyblue', label="MACD"),
    mpf.make_addplot(signal.loc['2023'], panel=2, color='gold', label="signal"),
    mpf.make_addplot(histogram_positive.loc['2023'], type='bar', width=0.7, panel=2, color='b'), 
    mpf.make_addplot(histogram_negative.loc['2023'], type='bar', width=0.7, panel=2, color='fuchsia'),
]

fig, ax = mpf.plot(df.loc[str(year)],
    type = "candle", style=style, addplot = add_plot,
    title = f"Market of stock {stock_code} in {year} year",
    ylabel = "price", ylabel_lower = 'volume',
    volume = True, figratio = (16, 9), returnfig=True
)
```

![cb450a6a-fa27-45e2-9b3d-52765b80d7c8](/cb450a6a-fa27-45e2-9b3d-52765b80d7c8.png)

## 与RSI对比

<a href="https://www.6young.site/blog/24009e7e.html" target="cardlink_"></a>

与RSI指标相比，RSI旨在表明相对于近期价格水平，市场是否被视为超买或超卖。RSI是一个振荡器，用于计算给定时间段内的平均价格收益和损失。默认时间段为 14 个时间段，值范围为 0 到 100。读数高于 70 表示超买情况，而低于 30 的读数被视为超卖，两者都可能表明顶部正在形成，反之亦然（底部正在形成）。

然而，MACD线没有像RSI和其他振荡器研究那样具体的超买/超卖水平。相反，它们在相对的基础上发挥作用。也就是说，投资者或交易者应该关注MACD/信号线的水平和方向与股票的先前价格走势的比较。

MACD、RSI等指标都衡量市场的动量，但由于它们衡量不同的因素，它们有时会给出相反的迹象。例如，RSI 可能会在持续一段时间内显示高于 70（超买），表明相对于近期价格，市场过度延伸到买方，而 MACD 表明市场的购买势头仍在增加。任一指标都可能通过显示与价格的背离来预示即将到来的趋势变化（价格继续走高，而指标转低，反之亦然）。

## MACD和确认的限制

SMA线背离的相关技术指标的主要问题之一是：它通常可以发出可能的反转信号，但随后没有发生实际的反转——它会产生**假阳性**；另一个问题是，背离并不能预测所有的反转。换句话说，它预测了太多没有发生的反转，而且错过了很多真实的价格反转。

这表明应通过趋势跟踪指标来确认未来的反转是否真实有效，例如方向运动指数（DMI）及平均方向指数 （ADX）。ADX旨在指示趋势是否到位，读数高于25表示趋势已经到位（在任一方向上），读数低于20表示没有趋势。

遵循 MACD 交叉和背离的投资者应在对 MACD 信号进行交易之前与 ADX 仔细检查。例如，虽然 MACD 可能显示出看跌背离，但检查 ADX 可能会告诉您趋势走高——在这种情况下，您将避免看跌 MACD 交易信号，并等待未来几天市场如何发展。

另一方面，如果 MACD 显示看跌交叉，而 ADX 处于非趋势区域 （<25），并且可能自行显示峰值和反转，您可能有充分的理由进行看跌交易。

此外，当资产价格在盘整中横盘整理时，例如在趋势之后的区间或三角形形态中，通常会发生假阳性背离。价格动量放缓（横盘走势或缓慢趋势走势）将导致 MACD 远离其先前的极端并被吸引到零线，即使没有真正的反转。同样，仔细检查 ADX 以确定趋势是否到位，并在采取行动之前查看价格正在做什么。

## 背离示例

当MACD形成的高点或低点超过价格上相应的高点和低点时，称为背离。当 MACD 形成两个上升低点与价格的两个下降低点相对应时，就会出现看涨背离。当长期趋势仍然积极时，这是一个有效的看涨信号。

即使长期趋势为负，一些交易者也会寻找看涨背离，因为它们可以发出趋势变化的信号，尽管这种技术不太可靠。

（借鉴了外国分析师的图，他们绿涨红跌）

![image-20231018141854321](/image-20231018141854321.png)

当 MACD 形成一系列两个下跌高点，对应于价格的两个上升高点时，就形成了看跌背离。在长期看跌趋势中出现的看跌背离被认为是趋势可能继续的确认。

一些交易者会在长期看涨趋势中关注看跌背离，因为它们可以表明趋势疲软。然而，它不如看跌趋势期间的看跌背离可靠。

![image-20231018141902167](/image-20231018141902167.png)

## 快速上升或下降的示例

当MACD快速上升或下跌（短期移动平均线远离长期移动平均线）时，这是一个信号，表明证券超买或超卖，并将很快恢复到正常水平。交易者通常会将此分析与 RSI 或其他技术指标相结合，以验证超买或超卖情况。

![image-20231018142028200](/image-20231018142028200.png)

投资者使用 MACD 直方图的方式与使用 MACD 本身的方式相同，这种情况并不少见。正或负交叉、背离以及快速上升或下降也可以在直方图上识别。在决定在任何给定情况下哪个是最好的之前，需要一些经验，因为 MACD 上的信号与其直方图之间存在时序差异。

## 交易者如何使用移动平均收敛/发散（MACD）？

交易者使用 MACD 来识别股票价格趋势的方向或强度的变化。MACD 乍一看似乎很复杂，因为它依赖于额外的统计概念，例如指数移动平均线 （EMA）。但从根本上说，MACD可以帮助交易者检测股票价格的近期动量何时可能预示着其潜在趋势的变化。这可以帮助交易者决定何时进入、添加或退出头寸。

MACD 看涨背离是指 MACD 未达到新低的情况，尽管股票价格已达到新低。这被视为看涨交易信号，因此称为“积极/看涨背离”。如果发生相反的情况——股价达到新高，但MACD未能做到这一点——这将被视为看跌指标，并称为“负/看跌背离”。在这两种情况下，设置都表明走高/走低不会持续，因此查看其他技术研究非常重要，例如上面讨论的相对强弱指数（RSI）。
