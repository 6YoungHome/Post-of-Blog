---
title: 【编程学习】使用Python修改图片分辨率
tags:
  - Python
  - 有用的代码
categories:
  - 编程学习
  - Python
typora-root-url: 【编程学习】使用python修改图片分辨率
cover: /img/blog_img/2.png
abbrlink: f92d10f5
date: 2023-06-30 19:48:56
description: 使用Python修改图片分辨率
---



### **提前下载并引入需要用到的Python库**

​	注意下载时，PIL库的名称为Pillow

```python
# pip install Pillow

from PIL import Image
```



### **实现函数：**

```python
def transfer(infile, outfile, new_wh=None, new_r=1):
    '''
    infile: 待修改图片文件路径【./input.png】
    outfile: 图片文件保存路径【./output.png】
    new_wh: 新的图片分辨率（宽*高）【(1280, 720)】
    new_r: 新的图片分辨率为原图比例（宽*高）
    '''
    im = Image.open(infile)
    if not new_wh:
        new_wh = (int(i*new_r) for i in im.size)
    reim = im.resize(new_wh)
    reim.save(outfile)
```



### **代码测试：**

```python
if __name__ == '__main__':
    infile = "./input.png"
    outfile1 = "./output1.png"
    outfile2 = "./output2.png"
    transfer(infile, outfile1, new_wh=(480, 270))
    transfer(infile, outfile2, new_r=0.5)
```

**原文件**

![image-20230630203243809](image-20230630203243809.png)

**调整为480*270**

![image-20230630203300970](image-20230630203300970.png)

**调整为原文件的一半**

![image-20230630203320743](image-20230630203320743.png)
