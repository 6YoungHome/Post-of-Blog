---
title: 【量化金融】各种投资组合理论的Python实现
typora-root-url: 【量化金融】各种投资组合理论的Python实现
cover: /img/blog_img/6.png
description: 使用Python实现多种投资组合理论
tags:
  - 量化金融
  - Python
  - 投资组合
categories:
  - 量化金融
  - 量化模型
abbrlink: c1a4a466
date: 2023-07-01 16:59:09
---



## 投资组合理论

### Markowitz模型

**马科维茨模型的假设条件**：

1. 投资者在考虑每一次投资选择时，其依据是某一持仓时间内的证券收益的概率分布。
2. 投资者是根据证券的期望收益率估测证券组合的风险。
3. 投资者的决定仅仅是依据证券的风险和收益。
4. 在一定的风险水平上，投资者期望收益最大；相对应的是在一定的收益水平上，投资者希望风险最小。

根据以上假设，马可维兹确立了证券组合预期收益、风险的计算方法和有效边界理论，建立了资产优化配置的均值-方差模型：
目标函数：$min \ \sigma^2(r_p)=\sum \sum x_ix_jCov(r_i,r_j)$

其中：$r_p=∑x_ir_i$

限制条件（允许卖空）：$\sum(x_i)=1$

限制条件（不允许卖空）：$\sum(x_i)=1,x_i\geq0$

```python
import numpy as np
import matplotlib.pyplot as plt
import scipy.linalg as linalg
import ffn

class MeanVariance:
    def __init__(self,returns):
        '''输入收益率'''
        self.returns=returns
    
    def min_var(self,goalRet):
        '''最小化方差函数'''
        covs=np.array(self.returns.cov())
        means=np.array(self.returns.mean())
        L1=np.append(np.append(covs.swapaxes(0,1),[means],0),[np.ones(len(means))],0).swapaxes(0,1)
        L2=list(np.ones(len(means)))
        L2.extend([0,0])
        L3=list(means)
        L3.extend([0,0])
        L4=np.array([L2,L3])
        L=np.append(L1,L4,0)
        results=linalg.solve(L,np.append(np.zeros(len(means)),[1,goalRet],0))
        return (np.array([list(self.returns.columns),results[:-2]]))
    
    def cal_var(self,fracs):
        '''给定个资产比例，计算方差'''
        return (np.dot(np.dot(fracs,self.returns.cov()),fracs))

    def cal_mean(self,fracs):
        mean_risky=ffn.to_returns(self.returns).mean()
        assert len(fracs)==len(mean_risky)
        return (np.sum(np.multiply(mean_risky,np.array(fracs))))

    def frontier_curve(self):
        '''最小方差前沿线绘制'''
        goals=[x/500000 for x in range(-100,4000)]
        variances=list(map(lambda x: self.cal_var(self.min_var(x)[1,:].astype(np.float)),goals))
        plt.plot(variances,goals)
```

### Black-Litterman模型

主要参数：

P、Q：投资人经验判断

e.g. A的收益率为15%；B比D收益率高5%；B和C比A收益率低6%
$$
P=\begin{bmatrix} 
1 & 0 & 0 & 0 & 0 \\ 
0 & 1 & 0 & -1 & 0 \\
-1 & \frac12 & \frac12 & 0 & 0 \\
\end{bmatrix}\\
$$
$$
q=[15\%,5\%,6\%]^T
$$
主观判断相对于先验信息所占比重$\tau$

```python
import numpy as np
import pandas as pd
from numpy import linalg
class BlackLitterman:
    def __init__(self,returns,tau,P,Q):
        mu = returns.mean()
        sigma = returns.cov()
        pi1 = mu
        ts = tau * sigma
        Omega = np.dot(np.dot(P,ts), P.T)* np.eye(Q.shape[0])
        middle = linalg.inv(np.dot(np.dot(P,ts),P.T)+Omega)
        er=np.expand_dims(pi1,axis=0).T + np.dot(np.dot(np.dot(ts, P.T), middle),(Q-np.expand_dims(np.dot(P,pi1.T),axis=1)))
        posterirorSigma = sigma+ts-np.dot(ts.dot(P.T).dot(middle).dot(P),ts)
        self.er=er
        self.posterirorSigma=posterirorSigma

    def blmin_var(self,goalRet):
        means=np.array(self.er)
        covs=np.array(self.posterirorSigma)
        L1=np.append(np.append((covs.swapaxes(0,1)),[means.flatten()],0),[np.ones(len(means))],0).swapaxes(0,1)
        L2=list(np.ones(len(means)))
        L2.extend([0,0])
        L3=list(means)
        L3.extend([0,0])
        L4=np.array([L2,L3])
        L=np.append(L1,L4,0)
        results=linalg.solve(L,np.append(np.zeros(len(means)),[1,goalRet],0))
        return pd.DataFrame(results[:-2],
                            index=self.posterirorSigma.columns,columns=['p_weight'])
```



## 资本资产定价模型(CAPM)

$R_{it}-R_{ft}=\alpha_i+\beta_i(R_{mt}-R_{ft})+\epsilon_{it}$

###### 根据数据用statsmodel.api.OLS拟合即可



## Fama-French三因子模型

市场风险溢酬因子，市值因子，账面市值比因子。

理论模型：
$$
\mathbb{E}(R_{it})-R_{ft}=\alpha_i+b_i(\mathbb{E}(R_{mt})-R_{ft})+s_i\mathbb{E}(SMB_t)+h_i\mathbb{E}(HML_t)
$$

实证检验：
$$
R_{it}-R_{ft}=\alpha_i+b_i(R_{mt}-R_{ft})+s_iSMB_t+h_iHML_t+\epsilon_i
$$



$SMB_t:$  做空市值小的公司，做多市值大的公司的投资组合的收益率。

$市值(ME_{kt})=股票价格(P_{kt}) * 在外流通股数(Q_{kt})$取中位数定义高低市值公司。

$HML_t:$  做多高B/M的公司，做空低B/M的公司的投资组合的收益率。

$$
B/M \ ratio_{kt}=\frac{账面价值BE_{kt}}{市值ME_{kt}}
$$
账面价值可从财务报表数据库获取用

**最后根据数据用statsmodel.api.OLS拟合即可**
