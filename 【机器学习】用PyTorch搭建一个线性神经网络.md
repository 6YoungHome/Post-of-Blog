---
title: 【机器学习】用PyTorch搭建一个线性神经网络
typora-root-url: 【机器学习】用PyTorch搭建一个线性神经网络
description: 利用Python的PyTorch框架搭建初步搭建一个神经网络模型。
cover: /img/blog_img/15.png
tags:
  - 神经网络
  - pytorch
  - python
categories:
  - 编程学习
  - 神经网络
abbrlink: 4ca0cf16
date: 2023-09-21 10:24:37
---





## 模拟数据集

模拟一个$y = w_1x_1 + w_2x_2 + b + \epsilon$的数据集

```python
from torch.utils import data
import torch
def init_linear_data(w, b, nums_example):
    """模拟一个有噪声的多元线性回归数据"""
    X = torch.normal(0, 1, (nums_example, len(w)))
    y = torch.matmul(X, w) + b
    y = y + torch.normal(0, 0.01, (nums_example, len(w)))
    return X, torch.reshape(y, (-1, 1))

def load_array(data_arrays, batch_size, is_train=True):  #@save
    """构造一个PyTorch数据迭代器"""
    dataset = data.TensorDataset(*data_arrays)
    return data.DataLoader(dataset, batch_size, shuffle=is_train)

w = torch.tensor([2, -3.4])
b = 4.2
X, y = init_linear_data(w, b, 1000)

batch_size = 10
data_iter = load_array((X, y), batch_size)
```



## 构建全连接层神经网络

```python
from torch import nn
# 构建网络
net = nn.Sequential(nn.Linear(2, 1)) # 一层网络；两个输入；一个输出

# 初始化参数
nn.init.xavier_uniform_(net[0].weight) # Xavier初始化方法
net[0].bias.data.fill_(0) #  0初始参数

# 定义损失函数
loss = nn.MSELoss()

# 选择模型优化算法
optimzer = torch.optim.SGD(net.parameters(), lr=0.002)
```

```python
num_epochs = 20
loss_record = []

for epoch in range(num_epochs):
    for X, y in data_iter:
        l = loss(net(X) ,y)
        optimzer.zero_grad()
        l.backward()
        optimzer.step()
    l = loss(net(features), labels)
    loss_record.append(float(l))
    #print(f'epoch {epoch + 1}, loss {l:f}')

plt.plot(list(range(1, len(loss_record)+1)),loss_record)
plt.xticks(list(range(1, 21)))
plt.xlabel("epoch")
plt.ylabel("loss")

print(net[0].weight, net[0].bias)

# output:
# Parameter containing:
# tensor([[ 1.9986, -3.3975]], requires_grad=True) Parameter containing:
# tensor([4.1981], requires_grad=True)
# 与预设参数w,b基本一致
```

![png](/loss.png)

















