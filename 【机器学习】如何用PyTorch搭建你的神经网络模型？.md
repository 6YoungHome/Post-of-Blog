---
title: 【机器学习】如何用PyTorch搭建你的神经网络模型？
typora-root-url: 【机器学习】如何用PyTorch搭建你的神经网络模型？
description: 了解PyTorch框架中神经网络的基本组件
cover: /img/blog_img/14.png
tags:
  - 神经网络
  - pytorch
  - python
categories:
  - 编程学习
  - 神经网络
abbrlink: f30c5b9a
date: 2023-09-20 22:33:12
---





## 基础操作

### 自动求梯度

```python
x = torch.arange(4.0) # 生成一个张量tensor([0., 1., 2., 3.])
x.requires_grad_(True) # 前两行等价于x=torch.arange(4.0,requires_grad=True)
print(x.grad) # 默认梯度为是None

y = 2 * torch.dot(x, x) # 把x计算一下，给y求梯度。
y.backward()
print(x.grad) # tensor([ 0.,  4.,  8., 12.])
```

**注意** ：

```python
# 在默认情况下，PyTorch会累积梯度，我们需要清除之前的值
# 不清除时：
y = x.sum() 
y.backward()
print(x.grad) # tensor([ 1.,  5.,  9., 13.])
# 清除时：
x.grad.zero_() # 清楚x的梯度
y = x.sum()
y.backward()
print(x.grad) # tensor([1., 1., 1., 1.])
```

### 分离计算

有时，我们希望将某些计算移动到记录的计算图之外。 

例如，假设y为x的函数$y=f(x)=x^2$，而z则是作为y和x的函数计算的$z=f(x,y)=x*y=x^3$。 如果我们想计算z关于x的梯度，但希望将y视为一个常数， 只考虑到x在y被计算后发挥的作用。

此时可以定义一个新变量u，该变量与y具有相同的值， 但丢弃计算图中如何计算y的任何信息。 换句话说，梯度不会向后流经u到x。 因此，下面的反向传播函数计算$z=u*x$关于x的偏导数，同时将u作为常数处理， 而不是$z=x^3$关于x的偏导数。

```python
x.grad.zero_()
y = x * x
u = y.detach()
z = u * x

z.sum().backward()
print(x.grad) # tensor([0., 1., 4., 9.])
```



## 神经网络基本组件

### 自定义网络层

```python
# 自定义一个不需要参数的层（只需要继承基础层类并实现前向传播功能。）
class DiffMeanLayer(nn.Module):
    """
    一个去除矩阵均值的网络层
    """
    def __init__(self):
        super().__init__()

    def forward(self, X):
        return X - X.mean()
```

```python
# 自定义一个需要参数的层（只需要继承基础层类并实现前向传播功能。）
class diyLinear(nn.Module):
    """
    一个自定义的全连接层
    """
    def __init__(self, in_units, units):
        super().__init__()
        self.weight = nn.Parameter(torch.randn(in_units, units))
        self.bias = nn.Parameter(torch.randn(units,))
        
    def forward(self, X):
        linear = torch.matmul(X, self.weight.data) + self.bias.data
        return F.relu(linear)
```

### 网络本体

```python
net = nn.Sequential(
    nn.Linear(4, 8), # 全连接层
    nn.ReLU(), # ReLU激活函数
    nn.Dropout(0.05), # Dropout层，提高模型泛化能力，要在激活函数后
    nn.DiffMeanLayer(), 
    nn.Sigmoid(), # Sigmoid激活函数
    nn.diyLinear(8, 1)
)
# 按需要不断叠加即可，可以添加自定义层
```

#### 网络参数共享

```python
# 我们需要给共享层一个名称，以便可以引用它的参数
shared = nn.Linear(8, 8)
net = nn.Sequential(
    nn.Linear(4, 8), nn.ReLU(),
    shared, nn.ReLU(),
    shared, nn.ReLU(),
    nn.Linear(8, 1)
)
```

#### 网络参数初始化

```python
# 直接对层初始化
net[0].weight.data.normal_(0, 0.01) # 一个正态分布的初始参数
net[0].bias.data.fill_(0) # 0初始参数
nn.init.xavier_uniform_(net[0].weight) # Xavier初始化方法 
nn.init.constant_(m.weight, 1) # 初始化常数参数

# 如果是全连接层，初始化参数为标准正态分布
def init_weights(m):
    """如果是全连接层，初始化参数为标准正态分布""" # 对层初始化参数
    if type(m) == nn.Linear:
        nn.init.normal_(m.weight, std=0.01)
```



### 损失函数

```python
loss = nn.MSELoss()
loss = nn.AdaptiveLogSoftmaxWithLoss()
# ...
# 很多，看情况选择，要了解不同损失的优劣
```

### 优化器

```python
optimzer = torch.optim.SGD(net.parameters(), lr=0.002)

# 权重衰减（L2范数衰减）
lambd = 3
optimzer = torch.optim.SGD([
        {"params":net[0].weight,'weight_decay': lambd},
        {"params":net[0].bias}], lr=lr
    )

# 还有多种优化器可以使用
```

## 模型文件读写

### 张量

```python
# 张量（列表/字典）都可以
x = torch.arange(4)

# 保存
torch.save(x, 'x-file')
torch.save([x,x], 'x-list')
torch.save({"x": x}, 'x-dict')

# 读取
x2 = torch.load('x-file')
x_list = torch.load('x-list')
x_dict = torch.load('x-dict')
```

### 模型

```python
# 定义并初始化一个网络
net = nn.Sequential(nn.Linear(4, 1))
nn.init.xavier_uniform_(net[0].weight)
net[0].bias.data.fill_(0) 

X = torch.randn(size=(1, 4)) # 初始化一个数据矩阵
Y = net(X)

# 保存
torch.save(net.state_dict(), 'net.params')

# 读取
clone = nn.Linear(4, 1) # 构建一个同样结构的网络
clone.load_state_dict(torch.load('net.params')) # 加载参数
clone.eval()
```

## 使用GPU运算

这一步应该在你安装pytorch前就应该完成，具体如何安装与配置cuda这里就不多说了，网上有很多教程。

```python
def try_gpu(i=0):  #@save
    """如果存在，则返回gpu(i)，否则返回cpu()"""
    if torch.cuda.device_count() >= i + 1:
        return torch.device(f'cuda:{i}')
    return torch.device('cpu')

def try_all_gpus():  #@save
    """返回所有可用的GPU，如果没有GPU，则返回[cpu(),]"""
    devices = [torch.device(f'cuda:{i}')
             for i in range(torch.cuda.device_count())]
    return devices if devices else [torch.device('cpu')]

try_gpu(), try_gpu(10), try_all_gpus()
# (device(type='cuda', index=0), device(type='cpu'), [device(type='cuda', index=0)])
# 因为我的电脑只有一个gpu，所以try_gpu(10)返回cpu
```

多个gpu时，需要将gpu0中的数据复制至gpu1，才能相互计算。

```python
X = torch.ones(2, 3, device=try_gpu(0))
Y = torch.rand(2, 3, device=try_gpu(1))

# 相加：
Z = X.cuda(1)
Y + Z
```

### 用GPU训练模型

两个要点，把数据迁移至gpu，把模型迁移至gpu

```python
X = torch.rand(2, 3)
X = X.to("cuda")
# 或者直接 X = torch.ones(2, 3, device=try_gpu(0))
print(x.device) # cuda:0

net = nn.Linear(3, 1)
net = net.to("cuda")
print(list(net.parameters())[0].device) # cuda:0
```

```python
# 初始化计算试一下
net[0].weight.data.normal_(0,1)
net[0].bias.data.zero_()

net(X)
# tensor([[-0.2529],
#        [ 0.3019]], device='cuda:0', grad_fn=<AddmmBackward0>)
```

要注意，只有在比较大的数据上，GPU的又是才能体现，按照样例给的如此小的两个矩阵相乘，计算就很慢，需要十几秒。
