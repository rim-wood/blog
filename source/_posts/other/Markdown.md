---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/other/markdown/markdown.jpg
title: Markdown学习
date: 2016-12-08 15:30:02
toc: true
tags: 
- Markdown
categories:
- other
---
Markdown 是一种轻量级的「标记语言」，它的优点很多，目前也被越来越多的写作爱好者，撰稿者广泛使用。看到这里请不要被「标记」、「语言」所迷惑，Markdown 的语法十分简单。常用的标记符号也不超过十个，这种相对于更为复杂的HTML 标记语言来说，Markdown 可谓是十分轻量的，学习成本也不需要太多，且一旦熟悉这种语法规则，会有一劳永逸的效果。
<!--more-->
# Markdown学习
## 标题
  `#`表示一级标题，`##`表示二级标题，多级标题依次累积
## 列表
### 1.有序列表
在文本前加入数字`1.`,`2.`,`3.`即可

    1. 列表1
    2. 列表2
    2. 列表3
### 2.无序列表
在文本前加入`-`

    - 列表1
    - 列表2
    - 列表3
*注：`-`,`1.`和文本之间要保留一个字符的空格。*
## 换行与缩进
换行使用`</br>`或者使用空行表示换行。缩进使用半方大的空白`&ensp;`或`&#8194;`也可以用
全方大的空白`&emsp;`或`&#8195;`例如：
    
    &ensp;你好</br>&emsp;世界
显示：</br>&ensp;你好</br>&emsp;世界
## 链接
格式为:`[文本](链接)`,例如

    [google链接](https://www.google.com)
    
显示如：[google链接](https://www.google.com)
## 图片
格式为：`![](图片链接)`,例如
    
    ![](http://g3.ykimg.com/0130391F4555C2DEFE9F592DF60A8431D4D237-366F-F0C2-ED67-4475501D05FC)
    
显示为：</br>![](http://g3.ykimg.com/0130391F4555C2DEFE9F592DF60A8431D4D237-366F-F0C2-ED67-4475501D05FC)
## 分割线
使用 
    <pre>
    *** 三个星
    * * *
    - - - 三个带空格的中划线
    ___ 连续三个下划线
    __________ 多个下划线
    </pre>
## 区块引用
使用`>`,例如
    
    > 你好</br>
    > 世界</br>
    > I`m coder
显示：
> 你好</br>
> 世界</br>
> I`m coder

## 代码区块
单句代码可以使用' `` '(也就是键盘~这个按钮)</br>
区块使用标签`<pre>`和`<code>`嵌套，例如：
    
    `你好`
    <pre><code>这是一个代码区块。</code></pre>
    
显示如：</br>
    `你好`
    <pre><code>这是一个代码区块。</code></pre>
## 粗体和斜体
粗体的格式为`**文本**`，斜体的格式为`*文本*`例如:
    
    **你好**</br>
    *世界*
显示:</br>
**你好**</br>
*世界*
## 表格
格式如下:
    
    | dog | bird | cat |
    |---- |------|---- |
    | foo | foo  | foo |
    | bar | bar  | bar |
    | baz | baz  | baz |
    
显示如下

| dog | bird | cat |
|---- |------|---- |
| foo | foo  | foo |
| bar | bar  | bar |
| baz | baz  | baz |

格式如下:
    
    | dog | bird | cat |
    |:----|:----:|----:|
    | foo | foo  | foo |
    | bar | bar  | bar |
    | baz | baz  | baz |
    
显示如下

| dog | bird | cat |
|:----|:----:|----:|
| foo | foo  | foo |
| bar | bar  | bar |
| baz | baz  | baz |

 