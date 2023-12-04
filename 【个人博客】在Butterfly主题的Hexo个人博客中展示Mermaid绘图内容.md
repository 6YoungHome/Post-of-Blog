---
title: 【个人博客】在Butterfly主题的Hexo个人博客中展示Mermaid绘图内容
typora-root-url: 【个人博客】在Butterfly主题的Hexo个人博客中展示Mermaid绘图内容
description: 在Butterfly主题的Hexo个人博客中展示Mermaid绘图内容
cover: /img/blog_img/6.png
tags:
  - Hexo
  - Mermaid
  - Butterfly
categories:
  - 个人博客
abbrlink: 8bb8b19c
date: 2023-08-10 10:35:15
---





## 使用方法

Butterfly主题下，使用mermaid进行绘图并展示十分的方便，打开项目的`_config.butterfly.yml`文件，将`mermaid`下的`enable`处参数设置为`true`。

```yaml
# mermaid
# see https://github.com/mermaid-js/mermaid
mermaid:
  enable: true
  # built-in themes: default/forest/dark/neutral
  theme:
    light: default
    dark: dark
```

在你的博客markdown文档中，在`mermaid`和`endmermaid`标签中添加你的mermaid代码。

```markdown
{% mermaid %}
some mermaid statements
{% endmermaid %}
```

## 示例

```
{% mermaid %}
stateDiagram-v2
    [*] --> Active
state Active {
    [*] --> NumLockOff
    NumLockOff --> NumLockOn : EvNumLockPressed
    NumLockOn --> NumLockOff : EvNumLockPressed
    --
    [*] --> CapsLockOff
    CapsLockOff --> CapsLockOn : EvCapsLockPressed
    CapsLockOn --> CapsLockOff : EvCapsLockPressed
    --
    [*] --> ScrollLockOff
    ScrollLockOff --> ScrollLockOn : EvCapsLockPressed
    ScrollLockOn --> ScrollLockOff : EvCapsLockPressed
}
{% endmermaid %}
```

{% mermaid %}
stateDiagram-v2
    [*] --> Active


state Active {
    [*] --> NumLockOff
    NumLockOff --> NumLockOn : EvNumLockPressed
	NumLockOn --> NumLockOff : EvNumLockPressed

​	--

​    [*] --> CapsLockOff
​    CapsLockOff --> CapsLockOn : EvCapsLockPressed
​	CapsLockOn --> CapsLockOff : EvCapsLockPressed

​	--

​    [*] --> ScrollLockOff
​    ScrollLockOff --> ScrollLockOn : EvCapsLockPressed
​    ScrollLockOn --> ScrollLockOff : EvCapsLockPressed
}
{% endmermaid %}

## 注意事项

1. 用这种方法要注意在`{% mermaid %}`与`{% endmermaid %}`两个标签中的代码可能因为markdown语法原因变化格式，要将其修复为无格式的形式。

2. 一个及其不好的点是`markdown`中的`mermaid`格式与`hexo`中的`mermaid`不同，因此没有同时在`markdown`与`hexo`中同时展示绘图结果的方法。

## 其他Mermaid绘图方案

{% link 【个人博客】使用Mermaid在Markdown中绘制类图,6Young,https://www.6young.site/blog/669fe5a0.html  %}

{% link 【个人博客】使用Mermaid在Markdown中绘制流程图,6Young,https://www.6young.site/blog/df7c1bab.html  %}

{% link 【个人博客】使用Mermaid在Markdown中绘制甘特图,6Young,https://www.6young.site/blog/875b5912.html  %}

{% link 【个人博客】使用Mermaid在Markdown中绘制其他类型图,6Young,https://www.6young.site/blog/c5116410.html  %}

# 
