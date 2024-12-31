---
title: go中rune数据类型
date: 2022-05-04 15:35:00
tags:
categories: Go
cover: /img/go.png
---

<!--more-->

什么是rune,看下官方回答
```go
// rune is an alias for int32 and is equivalent to
// int32 in all ways. It is
// used, by convention, to distinguish character
// values from integer values.

// int32的别名，几乎在所有方面等同于int32
// 它用来区分字符值和整数值
```
type rune = int32 
<b>划重点：用它来区分字符值和整数值</b>

大家知道计算机在底层处理的都是零和一的数字，那么字符串也不另外。string字符串在转换成byte之后其实都是一个数值，我们知道常规的英文字符是ascii码是通过一个字节( 2^8 其实还有一位是不用的 )来存储，中国文字、日本文字常用文字就有4000+，通过2^8肯定表达不了，所有可以通过unicode来存储，占用2个字节。

```go
package main

import "fmt"

func main()  {
	fmt.Println("go-algorithm",len("go-algorithm"))
	fmt.Println("go算法",len("go算法"))
}
```

输出如下：
```
go-algorithm 12
go算法 8
```


为什么go算法的长度是8，不应该是6么？
这个就要讲到go语言的编码是按照UTF-8编码规则来的。

UTF-8 顾名思义，是一套以 8 位为一个编码单位的可变长编码。会将一个码位编码为 1 到 4 个字节：
```
U+ 0000 ~ U+  007F: 0XXXXXXX
U+ 0080 ~ U+  07FF: 110XXXXX 10XXXXXX
U+ 0800 ~ U+  FFFF: 1110XXXX 10XXXXXX 10XXXXXX
U+10000 ~ U+10FFFF: 11110XXX 10XXXXXX 10XXXXXX 10XXXXXX
```

我们再深究下为什么 “算法” 占用6个字节的长度
我们查询到这两个汉字的16进制值得区间在UTF-8的第三区段，那么在go的编码下会占用三个字符。

由于编码的原因我们如果按照一个个字节的方式去处理字符串会导致处理规则没办知道是按照一个字节处理、两个字节处理或者三个字节处理，处理不对输出的子字符串可能是乱码。
```
var str = "go算法"
fmt.Println(str[:4])
// 输出：go�
```

所以在go语言中引进了rune的概念。在我们对字符串进去处理的时候只需要将字符串通过range去遍历，会按照rune为单位自动去处理，极其便利。
```
var str = "go算法"
	for k, v := range str {
		fmt.Printf("v type: %T\n", v)
		fmt.Println(v,k)
	}

输出如下：
v type: int32
103 0
v type: int32
111 1
v type: int32
31639 2
v type: int32
27861 5
我们可以通过 31639去查询他代表的汉字就是“算”
```

其中返回的v变量类型int32
在 Go 1.9 版本中对rune的定义如下：
```
type byte = uint8
type rune = int32
```

通过上面的代码我们已经很清楚的知道rune类型实质其实就是int32，他是go语言内在处理字符串及其便捷的字符单位。它会自动按照字符独立的单位去处理方便我们在遍历过程中按照我们想要的方式去遍历。
另外一个使用场景就是：我们在处理字符串的时候可以通过map[rune]int类型方便的判断字符串是否存在，其中 rune表示字符的UTF-8的编码值，int表示字符在字符串中的位置(按照字节的位置)。