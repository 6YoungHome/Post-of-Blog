---
title: 【编程学习】降低pandas.DataFrame的大小
typora-root-url: 【编程学习】降低pandas.DataFrame的大小
cover: /img/blog_img/3.png
description: Python使用技巧：通过改变数据类型降低pandas.DataFrame的大小
tags:
  - Python
  - Pandas
  - 数据分析
  - 有用的代码
categories:
  - 编程学习
  - Python
abbrlink: d16a9541
date: 2023-07-01 00:23:20
---





# 函数实现

众所周知，Python是数据分析过程中最常用的语言，其中，Pandas是最受欢迎的工具之一。但是，由于Python语言的特征，数据变量对于**数据类型**的定义并不重视。因此，在Pandas的DataFrame中，每一列数据大多使用默认的`int64`或者`float32`等数据类型，会造成较大的空间浪费。

以下代码就可以根据数据特征决定每一列的数据类型，有效的减小数据的大小：

```python
def reduce_memory_usage(df, verbose=True, inplace=False):
    if not inplace:
        df = df.copy() # 防止修改原数据
    numerics = ["int8", "int16", "int32", "int64", "float16", "float32", "float64"]
    start_mem = df.memory_usage().sum() / 1024 ** 2
    for col in df.columns:
        col_type = df[col].dtypes
        if col_type in numerics:
            c_min = df[col].min()
            c_max = df[col].max()
            if str(col_type)[:3] == "int":
                if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                    df[col] = df[col].astype(np.int8)
                elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                    df[col] = df[col].astype(np.int16)
                elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                    df[col] = df[col].astype(np.int32)
                elif c_min > np.iinfo(np.int64).min and c_max < np.iinfo(np.int64).max:
                    df[col] = df[col].astype(np.int64)
            else:
                if (
                    c_min > np.finfo(np.float16).min
                    and c_max < np.finfo(np.float16).max
                ):
                    df[col] = df[col].astype(np.float16)
                elif (
                    c_min > np.finfo(np.float32).min
                    and c_max < np.finfo(np.float32).max
                ):
                    df[col] = df[col].astype(np.float32)
                else:
                    df[col] = df[col].astype(np.float64)
    end_mem = df.memory_usage().sum() / 1024 ** 2
    if verbose:
        print(
            "Mem. usage decreased to {:.2f} Mb ({:.1f}% reduction)".format(
                end_mem, 100 * (start_mem - end_mem) / start_mem
            )
        )
    return df
```



# 函数测试

```python
df = pd.read_csv('huge_data.csv')
df1_mem = df.memory_usage().sum()/(1024**2)
df2_mem = reduce_memory_usage(df).memory_usage().sum()/(1024**2)
print(f"初始数据框占用内存: {round(df1_mem,2)}MB")
print(f"处理后数据框占用内存: {round(df2_mem, 2)}MB")

# output:
# 初始数据框占用内存: 70.68MB
# Mem. usage decreased to 50.46 Mb (28.6% reduction) 
# 处理后数据框占用内存: 50.46MB
```

