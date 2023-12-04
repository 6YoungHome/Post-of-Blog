---
title: 【机器学习】用PyTorch搭建一个多层感知机进行图像分类
typora-root-url: 【机器学习】用PyTorch搭建一个多层感知机进行图像分类
description: 利用Python的PyTorch框架搭建一个多层感知机神经网络，进行图像分类
cover: /img/blog_img/16.png
tags:
  - 神经网络
  - pytorch
  - python
categories:
  - 编程学习
  - 神经网络
abbrlink: 460fe2d9
date: 2023-09-21 11:33:26
---



## 下载数据集

下载“fashion_mnist”数据集，进行分类

```python
import torchvision
from torch.utils import data
from torchvision import transforms

def load_data_fashion_mnist(batch_size, resize=None): 
    """下载Fashion-MNIST数据集，然后将其加载到内存中"""
    trans = [transforms.ToTensor()]
    if resize:
        trans.insert(0, transforms.Resize(resize))
    trans = transforms.Compose(trans)
    mnist_train = torchvision.datasets.FashionMNIST(
        root="./data", train=True, transform=trans, download=True)
    mnist_test = torchvision.datasets.FashionMNIST(
        root="./data", train=False, transform=trans, download=True)
    return (data.DataLoader(mnist_train, batch_size, shuffle=True,
                            num_workers=workers),
            data.DataLoader(mnist_test, batch_size, shuffle=False,
                            num_workers=workers))

batch_size = 256
train_iter, test_iter = load_data_fashion_mnist(batch_size)
```

## 相关工具函数

```python
# 计算预测准确率
def accuracy(y_hat, y): 
    """计算预测正确的数量"""
    if len(y_hat.shape) > 1 and y_hat.shape[1] > 1:
        y_hat = y_hat.argmax(axis=1)
    cmp = y_hat.type(y.dtype) == y
    return float(cmp.type(y.dtype).sum())

def evaluate_accuracy(net, data_iter):  #@save
    """计算在指定数据集上模型的精度"""
    if isinstance(net, torch.nn.Module):
        net.eval()  # 将模型设置为评估模式
    metric = np.array([0,0])  # 正确预测数、预测总数
    with torch.no_grad():
        for X, y in data_iter:
            metric += [accuracy(net(X), y), y.numel()]
    return metric[0] / metric[1]
```



## 构建全连接层神经网络

```python
num_inputs = 784
num_outputs = 10
num_epochs = 10

# PyTorch不会隐式地调整输入的形状。因此，定义了展平层（flatten），来调整网络输入的形状
net = nn.Sequential(
    nn.Flatten(),
    nn.Linear(784, 256),
    nn.ReLU(),
    nn.Linear(256, 10)
)

# 参数初始化
def init_weights(m):
    """如果是全连接层，初始化参数为标准正态分布"""
    if type(m) == nn.Linear:
        nn.init.normal_(m.weight, std=0.01)
net.apply(init_weights)

# 损失函数与优化器
lr, num_epochs = 0.1, 10
loss = nn.CrossEntropyLoss(reduction='none')
optimzer = torch.optim.SGD(net.parameters(), lr=lr)


```

```python
# 训练
data_ls = []
for epoch in trange(num_epochs):
    metric = np.array([0, 0, 0])
    for X, y in train_iter:
        # 计算梯度并更新参数
        y_hat = net(X)
        l = loss(y_hat, y)
        # 使用PyTorch内置的优化器和损失函数
        optimzer.zero_grad()
        l.mean().backward()
        optimzer.step()
        metric += [float(l.sum()), accuracy(y_hat, y), y.numel()]
    train_loss, train_acc = metric[0] / metric[2], metric[1] / metric[2]
    test_acc = evaluate_accuracy(net, test_iter)
    data_ls.append((test_acc, train_loss, train_acc))
    
fig, ax = plt.subplots(1, 1)
ax.plot(list(range(len(data_ls))), [i[1] for i in data_ls], label="train_loss")
ax.plot(list(range(len(data_ls))), [i[2] for i in data_ls], label="train_acc")
ax.plot(list(range(len(data_ls))), [i[0] for i in data_ls], label="test_acc")
ax.legend()
```

![image-20230921113032647](/image-20230921113032647.png)

















