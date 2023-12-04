---
title: 【个人博客】Hexo中特别好用的插件有哪些
typora-root-url: 【个人博客】Hexo中特别好用的插件有哪些
cover: /img/blog_img/14.png
description: Hexo博客画图，绘制思维导图，Echarts图以及卡片式链接
tags:
  - Hexo
  - Echart
  - markmap
  - 思维导图
  - 卡片式链接
categories:
  - 个人博客
abbrlink: 94b613b0
date: 2023-07-10 14:45:35
---



20230805更新~~~



---

# Echarts动态图表

[hexo-tag-echarts3](https://www.npmjs.com/package/hexo-tag-echarts3)是一款基于 [Echarts](https://echarts.apache.org/zh/index.html) 的动态图表插件。

通过`npm install hexo-tag-echarts3 --save`下载插件。

由于原作者已不再维护，echarts 的版本较老，可以修改`./node_modules/hexo-tag-echarts3/template.html`文件中的js引用路径：

```html
- <script src="https://cdn.bootcss.com/echarts/3.8.0/echarts.common.min.js"></script>
+ <script src="https://cdn.bootcss.com/echarts/5.2.1/echarts.common.min.js"></script>
```

- 修改 echarts 到最新版本后，语法也会相应地改变，请到[官网示例](https://echarts.apache.org/examples/zh/index.html)自行查询。选择一个需要的图表，调整参数后拷贝`option=`后面的部分并粘贴到`echarts`标签内。



- 在markdown中直接使用的代码为：

```markdown
{% echarts [height] [weight] %}

echarts 配置

{% endecharts %}
```

例如：

{% echarts 400 '81%' %}
{
  tooltip: {
    trigger: 'axis',
    axisPointer: {
      type: 'shadow'
    }
  },
  toolbox: {
    show: true,
    feature: {
      dataZoom: {
        yAxisIndex: 'none'
      },
      magicType: { type: ['line', 'bar','stack'] },
      restore: {},
      saveAsImage: {}
    }
  },
  legend: {
    data: ['Profit', 'Expenses', 'Income']
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '3%',
    containLabel: true
  },
  xAxis: [
    {
      type: 'value'
    }
  ],
  yAxis: [
    {
      type: 'category',
      axisTick: {
        show: false
      },
      data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    }
  ],
  series: [
    {
      name: 'Profit',
      type: 'bar',
      label: {
        show: true,
        position: 'inside'
      },
      emphasis: {
        focus: 'series'
      },
      data: [200, 170, 240, 244, 200, 220, 210]
    },
    {
      name: 'Income',
      type: 'bar',
      stack: 'Total',
      label: {
        show: true
      },
      emphasis: {
        focus: 'series'
      },
      data: [320, 302, 341, 374, 390, 450, 420]
    },
    {
      name: 'Expenses',
      type: 'bar',
      stack: 'Total',
      label: {
        show: true,
        position: 'left'
      },
      emphasis: {
        focus: 'series'
      },
      data: [-120, -132, -101, -134, -190, -230, -210]
    }
  ]
};
{% endecharts %}

是通过以下代码构建并显示的：

```markdown
{% echarts 400 '81%' %}
{
  tooltip: {
    trigger: 'axis',
    axisPointer: {
      type: 'shadow'
    }
  },
  toolbox: {
    show: true,
    feature: {
      dataZoom: {
        yAxisIndex: 'none'
      },
      magicType: { type: ['line', 'bar','stack'] },
      restore: {},
      saveAsImage: {}
    }
  },
  legend: {
    data: ['Profit', 'Expenses', 'Income']
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '3%',
    containLabel: true
  },
  xAxis: [
    {
      type: 'value'
    }
  ],
  yAxis: [
    {
      type: 'category',
      axisTick: {
        show: false
      },
      data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    }
  ],
  series: [
    {
      name: 'Profit',
      type: 'bar',
      label: {
        show: true,
        position: 'inside'
      },
      emphasis: {
        focus: 'series'
      },
      data: [200, 170, 240, 244, 200, 220, 210]
    },
    {
      name: 'Income',
      type: 'bar',
      stack: 'Total',
      label: {
        show: true
      },
      emphasis: {
        focus: 'series'
      },
      data: [320, 302, 341, 374, 390, 450, 420]
    },
    {
      name: 'Expenses',
      type: 'bar',
      stack: 'Total',
      label: {
        show: true,
        position: 'left'
      },
      emphasis: {
        focus: 'series'
      },
      data: [-120, -132, -101, -134, -190, -230, -210]
    }
  ]
};
{% endecharts %}

```



# Markmap思维导图

## 思维导图创建

方案源于：

{% link Hexo 的思维导图插件,猎人杂货铺,https://hunterx.xyz/hexo-simple-mindmap-plugin-intro.html  %}



[hexo-markmap](https://www.npmjs.com/package/hexo-markmap)是一款轻量的思维导图插件，在`pullquote`标签内使用 markdown 语言可生成对应的思维导图。

使用`npm install hexo-simple-mindmap --save`安装插件

在markdown中直接使用的代码为：

```markdown
{% pullquote mindmap mindmap-md %}

相关内容

{% endpullquote %}
```

{% pullquote mindmap mindmap-md %}
- 知识图谱
  - Scala
  - Python
  - java
      - jvm
  - 消息队列
      - kafka
      - pulsar
          - test1
              - test2

{% endpullquote %}

- 上述思维导图是通过以下代码构建并实现：

```markdown
{% pullquote mindmap mindmap-lg %}

- 知识图谱
    - Scala
    - Python
    - java
      - jvm
    - 消息队列
      - kafka
      - pulsar
        - test1
          - test2

{% endpullquote %}
```

## 插件缺陷

1. 插件不支持 Mathjax 公式渲染
2. 思维导图不能放大
3. 思维导图只能有一个根节点
4. 有些主题下导图显示有问题（下方有解决方案）

## 思维导图显示问题

- 可能有人生成思维导图后，幕布只会占据屏幕的一半。

![image-20230711005039833](/image-20230711005039833.png)

- 通过检查页面的源代码，发现是blockquote的css设置有问题，此处的max-width被设置成了45%。

![image-20230711005358236](/image-20230711005358236.png)

- 由于我查了好久，也没找到设置这一样式的源文件，因此选择另辟蹊径，自定义一个css样式文件处理这个问题。在`[Blogroot]\themes\butterfly\source\css`路径下新建`custom.css`（不强制，都可以）文件，在其中写入代码：

```css
blockquote.pullquote {
  position: relative;
  max-width: 100%;
  font-size: 110%;
}
```

- 修改`[Blogroot]\_config.butterfly.yml`文件，引入自定义的css样式文件

```yaml
inject:
  head:
    - <link rel="stylesheet" href="/css/custom.css">
```



# 卡片式引用链接

卡片式的引用链接可以让你的引用更加显眼，也让博客更加好看美观，我在网上查了很多方法，最后都不能用，只有一个方案成功实现，给大家分享一下，方案源于：

{% link 使用 CardLink 库生产卡片式链接,Lete乐特,https://blog.imlete.cn/article/CardLink.html %}

首先，在_config.butterfly.yml的Inject处添加如下代码：

```yaml
inject:
  head:
    - <script src="https://cdn.jsdelivr.net/npm/cardlink"></script>
  bottom:
    - <script>cardLink(document.querySelectorAll('a[target=cardlink_]'))</script>
```

- 此处的“cardlink_”可以进行自定义，添加一个独特的标签即可，然后可以在markdown中直接键入html代码（不加代码框）

```html
<a href="https://www.6young.site/blog/1f6043a5.html" target="cardlink_"></a>
```

然后就能看到卡片式链接了：

{% link 【个人博客】将个人博客部署至Vercel上,6Young,https://www.6young.site/blog/1f6043a5.html %}

欢迎在下方评论区分享讨论！





