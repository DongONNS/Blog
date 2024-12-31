---
title: markdown常用语法
date: 2019-10-15 21:51:46
tags:
categories: 实用操作
---

如何用markdown写博客？需要的语法都在这里

<!--more-->

># **标题**


	# 一级标题

    ## 二级标题

    ### 三级标题

    #### 四级标题

    ##### 五级标题
<br>

# 一级标题

## 二级标题

### 三级标题

#### 四级标题

##### 五级标题

---
<br>


># **引用**

    >一级引用
    >
    >>二级引用
    >
    >一级引用

>一级引用
>
>>二级引用
>
>一级引用

---
<br>


># **行开头加“点”**

    * red

    * green

    * blue

* red

* green

* blue

---
<br>

># **加上序号**

    1.red
    
    2.green
    
    3.red

1.red

2.green

3.red

---
<br>

># **代码块区域：直接点上面的code“<>”就可以了；** #

    public class Hello{
    	public static void main(String[] args){
    		System.out.println("Hello MarkDown);
    	}	
    }

---
<br>

># **斜体**

    *这是斜体的方式*
    
    _这是斜体的方_


*这是斜体的方式*

_这是斜体的方_

---
<br>

># **粗体**

    **这是粗体**
    
    __这是粗体__


**这是粗体**

__这是粗体__

---
<br>
># **标记行内小段代码**
>加上 ‘’ 即可

    use the `printf()` function.

use the `printf()` function.

---
<br>

># **自动连接**

    语法格式：加上左右尖括号
    <http://example.com/>


*示例：字节跳动招聘官网*

<https://job.bytedance.com/intern>

---
<br>

># **行内链接**

    语法格式
    
    这是 [字节跳动](https://job.bytedance.com/intern/ "title") 的连接.

这是[字节跳动](https://job.bytedance.com/intern/ "title")的连接

---
<br>
<br>

># **图片**

    语法格式：
    
    [图片alt](图片地址 ''图片title'')
    
    图片alt就是显示在图片下面的文字，相当于对图片内容的解释。
    
    图片title是图片的标题，当鼠标移到图片上时显示的内容。title可加可不加
	
    ![字节跳动](https://wx1.sinaimg.cn/mw690/007857NYgy1g7z99tuyqbj30hs0hsglq.jpg)
    
![字节跳动](https://wx1.sinaimg.cn/mw690/007857NYgy1g7z99tuyqbj30hs0hsglq.jpg)

---

># <div align=left>**如何实现首行缩进**

    &ensp;这是半方大的空白
    &emsp;这是全方大的空白
    &nbsp;这是不断行的空白

<div align=left>&ensp;这是半方大的空白

<div align=left>&emsp;这是全方大的空白

<div align=left>&nbsp;这是不断行的空白

<br>
---

># **如何设置图片居中**

在图片标签之前加上`<div align=center>`标签

    <div align=center>![字节跳动](https://wx1.sinaimg.cn/mw690/007857NYgy1g7z99tuyqbj30hs0hsglq.jpg)

<div id=1 align=center>![字节跳动](https://wx1.sinaimg.cn/mw690/007857NYgy1g7z99tuyqbj30hs0hsglq.jpg)

---
<br>
<br>

># **表格**

![表格的表达方式](https://wx3.sinaimg.cn/small/007857NYly1g8z372kcgrj305104r3ya.jpg)

|表头|表头|表头|
|:---|:--:|:---:
|内容|内容|内容|
|内容|内容|内容|
    
    第二行分割表头和内容。
    - 有一个就行，为了对齐，多加了几个
    文字默认居左
    -两边加：表示文字居中
    -右边加：表示文字居右
    注：原生的语法两边都要用 | 包起来。


---
<br>

># **流程图**

此处留白先，不会用。

<br>
># **分割线**

    `---`

---
<br/>

># **转义符号**
>\   下面这些符号可以通过转义符号来实现

    
    \   反斜线
    
    `   反引号
    
    *   星号
    
    _   底线
    
    {}  花括号
    
    []  方括号
    
    ()  括弧
    
    #   井字号
    
    +   加号
    
    -   减号
    
    .   英文句点
    
    !   惊叹号


---




