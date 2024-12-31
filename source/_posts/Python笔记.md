---
title: Python笔记
date: 2019-10-26 16:40:25
categories: Python笔记
tags:
---

<!--more-->

### 在print(r'Hello,Python')中的r是什么意思？

r代表的是原始字符串标识符,也就是说 用r' '表示' '内部的字符串默认不转义

## 输出：print()
    eg.print('The quick brome fox','jumps over','the lazy dog')
    The quick brome fox jumps over the lazy dog

## 输入：input()

上述命令即可进行人机交互了
    eg.name=input()
       Daniel

可以在input里面放入一些提示信息如下所示

    name=input('please input your name:')
    print('hello,',name)

运行上面的代码结果如下所示

    please input your name:Daniel
    hello Daniel

## 字符编码

对于单个字符，Python提供了ord（）函数获取字符的整数表示和chr()函数把编码转换成对应的字符：
    
    >>>ord('A')
    65
    >>>ord('中')
    20013
    >>>chr(66)
    'B'
    >>>chr(25991)
    '文'

如果知道字符的整数编码，还可以用十六进制这么写

    >>> '\u4e2d\u6587'
    '中文'

要注意区分'ABC'和b'ABC',两者内容上是一样的，但是str类型的数据像'ABC'，它在内存中是以Unicode表示，一个字符对应若干字节，而b'ABC'每个字符只占一个字节，以Unicode表示的str通过encode()方法可以编码为指定的bytes,例如：

    >>> 'ABC'.encodes('ascii')
    b'ABC'

英文的str可以用ASCII编码为bytes，内容是一样的，含有中文的str可以用UTF-8编码为bytes。含有中文的str无法用ASCII编码，因为中文编码的范围超过了ASCII编码的范围，Python会报错。

反过来，如果我们从网络或磁盘上读取了字节流，那么读到的数据就是bytes。要把bytes变为str，就需要用**decode()**方法：

    >>> b'\xe4\xb8\xad\xe6\x96\x87'.decode('utf-8')
    '中文'

如果bytes中包含无法解码的字节，decode()方法会报错：

    >>> b'\xe4\xb8\xad\xff'.decode('utf-8')
    Traceback (most recent call last):
      ...
    UnicodeDecodeError: 'utf-8' codec can't decode byte 0xff in 

    position 3: invalid start byte


如果bytes中只有一小部分无效的字节，可以传入errors='ignore'忽略错误的字节：

    >>> b'\xe4\xb8\xad\xff'.decode('utf-8', errors='ignore')
    '中'

要计算str包含多少个字符，可以用len()函数：

    >>> len('ABC')
    3
    >>> len('中文')
    2
len()函数计算的是str的字符数，如果换成bytes，len()函数就计算字节数：

    >>> len(b'ABC')
    3
    >>> len(b'\xe4\xb8\xad\xe6\x96\x87')
    6
    >>> len('中文'.encode('utf-8'))
    6

可见，1个中文字符经过UTF-8编码后通常会占用3个字节，而1个英文字符只占用1个字节。

在操作字符串时，我们经常遇到str和bytes的互相转换。为了避免乱码问题，应当**始终坚持使用UTF-8编码**对str和bytes进行转换。

当Python解释器读取源代码时，为了让它按UTF-8编码读取，我们通常在文件开头写上这两行：

    #!/usr/bin/env python3
    # -*- coding: utf-8 -*-

第一行注释是为了告诉Linux/OS X系统，这是一个Python可执行程序，Windows系统会忽略这个注释；

第二行注释是为了告诉Python解释器，按照UTF-8编码读取源代码，否则，你在源代码中写的中文输出可能会有乱码。

申明了UTF-8编码并不意味着你的.py文件就是UTF-8编码的，必须并且要确保文本编辑器正在使用UTF-8 without BOM编码

在Python中，采用的格式化方式和C语言是一致的，用%实现，举例如下：

    >>> 'Hello, %s' % 'world'
    'Hello, world'
    >>> 'Hi, %s, you have $%d.' % ('Michael', 1000000)
    'Hi, Michael, you have $1000000.'

  占位符    	替换内容
   %d      	 整数
   %f	    浮点数
   %s	    字符串
   %x	  十六进制整数

如果你不太确定应该用什么，%s永远起作用，它会把任何数据类型转换为字符串：

    >>> 'Age: %s. Gender: %s' % (25, True)
    'Age: 25. Gender: True'

有些时候，字符串里面的%是一个普通字符怎么办？这个时候就需要转义，用%%来表示一个%：

    >>> 'growth rate: %d %%' % 7
    'growth rate: 7 %

## format()
另一种格式化字符串的方法是使用字符串的format()方法，它会用传入的参数依次替换字符串内的占位符{0}、{1}……，不过这种方式写起来比%要麻烦得多：

    >>> 'Hello, {0}, 成绩提升了 {1:.1f}%'.format('小明', 17.125)
    'Hello, 小明, 成绩提升了 17.1%'

