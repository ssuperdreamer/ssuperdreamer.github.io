---
layout: post
title: "String in Swift"
subtitle: "Swift 中的String"
date: 2021-04-16 14:40:10
author: "Axag"
header-img: "img/post-bg-rwd.jpg"
tags: [iOS百宝箱]
---
# Swift 中的String

![20210416_1](/img/2021/20210416_1.png)

接下来所有的例子都使用该字符串

``` var str = "Hello, playground"```

在swift4中 Strings做了一个重大的修改，当你从一个String中做substring操作的时候，你得到的是一个Substring的类型而不是String，为什么会这样呢？ **在swift中String是值类型.** 这意味着如果你用一个String来创建一个新的String, 那么这将会是一个拷贝. 这样有利于代码的稳定(不会有在你知道不知道的情况下发生变动)但是相对的效率就会差些

换一个角度说，SubString是原始String的一个引用拷贝，下图是来自[文档](https://docs.swift.org/swift-book/LanguageGuide/StringsAndCharacters.html)里的可以证实这一点

![20210416_2](/img/2021/20210416_2.png)
没有拷贝的时候的确能更高效的使用,但是，假设你有10个字符的SubString从一个百万长度的字符串，因为**SubString**是引用自String的,那么只要SubString存在,那么系统就必要持有整个的字符串. 因此，无论何时操作完子字符串，都要将其转换为字符串。

```let myString = String(mySubstring)```

这将只复制**subString**，并且可以回收保存旧String的内存。**SubString**（作为一个类型）的寿命很短。

Swift4中另一个重大的改变是**Strings**是一个**Collections**. 这意味着你对Collection做操作也可以用在String上（使用下标、对字符进行迭代、筛选等）。

下面来举些例子说下在**Swift**中如果获得**SubString**

## 获得substrings

你可以使用下标或许多其他方法（例如，prefix、suffix、split）从**String**中获取**SubString**。你还需要使用**Sting.Index**但不是范围的 **Int**索引.

### 从头部操作String

使用下标(subscbscript):

```swift
let index = str.index(str.startIndex, offsetBy: 5)
let mySubstring = str[..<index] 
```

prefix:

```swift
let index = str.index(str.startIndex, offsetBy: 5)
let mySubstring = str.prefix(upTo: index) // Hello
```

或者更简单

```swift
let mySubstring = str.prefix(5) // Hello
```

### 从尾部操作String

使用subscbscript:

```swift
let index = str.index(str.endIndex, offsetBy: -10)
let mySubstring = str[index...] // playground
```

suffix:

```swift
let index = str.index(str.endIndex, offsetBy: -10)
let mySubstring = str.suffix(from: index) // playground
```

或者更简单

```swift
let mySubstring = str.suffix(10) // playground
```

注意，当使用后缀（from:index）时，需要使用-10从结尾进行计数。当只使用后缀（x）时，就不用了，只要字符串的最后x个字符。

### 从范围操作 String

使用下标(subscripts):

```swift
let start = str.index(str.startIndex, offsetBy: 7)
let end = str.index(str.endIndex, offsetBy: -6)
let range = start..<end
let mySubstring = str[range]  // play
```

### 将 SubString转为String

别忘了，当你准备好保存你的子字符串时，你应该把它转换成一个字符串，这样旧字符串的内存就可以被清除了。

```swift
let myString = String(mySubstring)
```
