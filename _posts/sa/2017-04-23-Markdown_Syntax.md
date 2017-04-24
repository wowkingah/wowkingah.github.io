---
layout: post
title: Markdown常用语法
date: 2017-04-23 
tags: markdown
---

[TOC]  

### 什么是 Markdown
Markdown 是一种方便记忆、书写的纯文本标记语言，用户可以使用这些标记符号以最小的输入代价生成极富表现力的文档。  

### 标题 
一级标题：`# Header 1`  
二级标题：`## Header 2`           
三级标题：`### Header 3`           
四级标题：`#### Header 4`           
五级标题：`##### Header 5`            
六级标题：`###### Header 6`  
![标题](/images/posts/markdown_syntax/header.jpg)

### 字体
斜体：`*斜体*`  
粗体：`**粗体**`  
粗斜体：`***粗斜体***`  
删除线：`~~删除线~~`  
![字体](/images/posts/markdown_syntax/font.jpg)

### 目录
在段落中填写 `[TOC]` 以显示全文内容的目录结构。
![目录](/images/posts/markdown_syntax/catalogue.jpg)

### 段与行
段落：`段落之间空一行`  
换行：`一行结束时输入两个空格`  

### 列表项
列表项1：`* 列表项1`  
列表项2：`* 列表项2`  
列表项3：`+ 列表项3`  
列表项4：`+ 列表项4`  
列表项5：`+ 列表项5`  
列表项6：`+ 列表项6`  
列表项7：`1. 列表项7`  
列表项8：`2. 列表项8`  
列表项9：`- [ ] 列表项9`  
列表项10：`- [x] 列表项10`  
![列表项](/images/posts/markdown_syntax/list.jpg)

### 引用
引用：`> 引用内容，可结合列表项一起使用`  
![引用](/images/posts/markdown_syntax/reference.jpg)

### 表格
`| 项目 | 价格 | 数量 |`
`| -------- | -----: | :----: |`
`| 计算机 | 8000 | 10 |`
`| 手机 | 6000 | 5 |`
`| 电视 | 3000 | 1 |`
![表格](/images/posts/markdown_syntax/table.jpg)

### 链接与图片
链接：`[Title](URL)`  
图片：`![](/images/image.jpg)`  
![链接](/images/posts/markdown_syntax/link.jpg)

### 代码高亮
\```python
    print 'hello world~'
\```
![代码](/images/posts/markdown_syntax/code.jpg)

### 水平线
水平线：`---`  
![水平线](/images/posts/markdown_syntax/horizontal_rules.jpg)  

### 注脚
使用 [^keyword] 表示注脚。
这是一个*注脚*[^footnote]的样例。

### 定义型列表

名词 1
:   定义 1（左侧有一个可见的冒号和四个不可见的空格）

代码块 2
:   这是代码块的定义（左侧有一个可见的冒号和四个不可见的空格）

        代码块（左侧有八个不可见的空格）

### HTML标签
支持在 Markdown 语法中嵌套 Html 标签，譬如，你可以用 Html 写一个纵跨两行的表格：

    <table>
        <tr>
            <th rowspan="2">值班人员</th>
            <th>星期一</th>
            <th>星期二</th>
            <th>星期三</th>
        </tr>
        <tr>
            <td>李强</td>
            <td>张明</td>
            <td>王平</td>
        </tr>
    </table>

<table>
    <tr>
        <th rowspan="2">值班人员</th>
        <th>星期一</th>
        <th>星期二</th>
        <th>星期三</th>
    </tr>
    <tr>
        <td>李强</td>
        <td>张明</td>
        <td>王平</td>
    </tr>
</table>


### reference
https://www.zybuluo.com/mdeditor?url=https://www.zybuluo.com/static/editor/md-help.markdown
  
  
[^footnote]: 这是一个 *注脚* 的文本。
