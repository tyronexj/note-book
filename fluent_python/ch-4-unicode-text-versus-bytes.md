# 一文读懂 Python 中的 Unicode

## Unicode 解决了什么问题？

[ASCII 码表](https://www.asciitable.com)包含了常用的 128 个字符(包括 A-Za-z 的英文字符)，在计算机中，他占用一个字节(byte)的大小。1 个 byte 有 8 个 bit，最多只能表示 256 个字符。

但是这个世界上还有很多其他的语言，例如，中文，日文，法文等，它们的字符集一般都超过 256 个，这样就无法容纳在 1 个 byte 中，因此需要扩展编码的大小。

在 Unicode 出现之前，编码体系比较混乱，例如

- 微软的[cp1252](https://zh.wikipedia.org/wiki/Windows-1252)
- IBM 的[cp437](https://en.wikipedia.org/wiki/Code_page_437)
- 简体中文的[gb2312](https://zh.wikipedia.org/wiki/GB_2312)

这就导致了两个问题：

1. 没有一个编码体系可以容纳所有的字符
1. 编码体系不统一，导致了很多乱码和兼容性问题

**Unicode 标准的制定，就是为了统一编码体系。就像秦始皇统一度量衡一样。**

## Unicode 码的数值和名字

Unicode 的编码值从`U+0000`到`U+10FFFF`，共有`1,114,112`个码位。其中，`U+0000`到`U+FFFF`称为基本多文种平面(BMP)，`U+10000`到`U+10FFFF`称为辅助多文种平面(SMP)。

- 查看一个字符对应的 Unicode 编码值

```python
>>> ord('e')
101
>>> ord('é')
233
>>> ord('中')
20013
```

- 查看一个 Unicode 编码值对应的字符

```python
>>> chr(101)
'e'
>>> chr(233)
'é'
>>> chr(20013)
'中'
```

- 查看一个字符对应的 Unicode 编码名字

```python
>>> from unicodedata import name
>>> name('e')
'LATIN SMALL LETTER E'
>>> name('é')
'LATIN SMALL LETTER E WITH ACUTE'
>>> name('中')
'CJK UNIFIED IDEOGRAPH-4E2D'
>>> name('☺')
'WHITE SMILING FACE'
```

- 根据 Unicode 编码名字查看对应字符

```
>>> u'\N{LATIN SMALL LETTER E}'
'e'
>>> u'\N{LATIN SMALL LETTER E WITH ACUTE}'
'é'
>>> u'\N{CJK UNIFIED IDEOGRAPH-4E2D}'
'中'
>>> u'\N{WHITE SMILING FACE}'
'☺'
```

## Unicode 的编码和解码

读到这里，你可能会问，Unicode 已经统一了编码体系，为什么还需要编码和解码呢？

如果直接把 Unicode 码值进行 IO 处理（写入文件，发送至网络），那么 IO 的大小就会变得很大，因为一个 Unicode 码值占用 4 个字节，而大多数的文件可能只包含英文字母，而一个英文字符只需要 1 个字节。参考上述 ASCII 码表。

为了解决 IO 的大小问题，Unicode 提供了两种编解码方法：

- utf-8
- utf-16

```python
>>> 'c'.encode('utf-8')
b'c'
>>> b'\x63'.decode('utf-8')
'c'
>>> '中'.encode('utf-8')
b'\xe4\xb8\xad'
>>> b'\xe4\xb8\xad'.decode('utf-8')
'中'
```

我们看到，在 uft-8 编码模式下，一个英文字符被编码为 1 个字节，一个中文字符被编码为 3 个字节。如果没有编码，那么所有字符的 IO 大小都将会是 4 个字节。

一个稍微复杂一点的程序来比较各种编码结果(可跳过此段代码，直接查看结果)

```python
codes = ['e', 'é', 'Ã', 'Ω', '“', '€', '中', '气', '氣', '𝄞']
codecs = ['ascii', 'latin1', 'cp1252', 'cp437', 'gb2312', 'utf-8', 'utf-16be']

print("|char|unicode|", end='')
for codec in codecs:
    print(codec, end='|')
print(end='\n')

print("---".join(["|"] * (len(codecs) + 3)))

for code in codes:
    print(f"|{code}|U+{ord(code):04X}|", end='')
    for codec in codecs:
        bcode = code.encode(codec, errors='replace')
        print(" ".join(f"{c:02X}" if c != 0x3F else '*' for c in bcode), end='|')
    print(end='\n')
```

| char | unicode | ascii | latin1 | cp1252 | cp437 | gb2312 | utf-8       | utf-16be    |
| ---- | ------- | ----- | ------ | ------ | ----- | ------ | ----------- | ----------- |
| e    | U+0065  | 65    | 65     | 65     | 65    | 65     | 65          | 00 65       |
| é    | U+00E9  | \*    | E9     | E9     | 82    | A8 A6  | C3 A9       | 00 E9       |
| Ã    | U+00C3  | \*    | C3     | C3     | \*    | \*     | C3 83       | 00 C3       |
| Ω    | U+03A9  | \*    | \*     | \*     | EA    | A6 B8  | CE A9       | 03 A9       |
| “    | U+201C  | \*    | \*     | 93     | \*    | A1 B0  | E2 80 9C    | 20 1C       |
| €    | U+20AC  | \*    | \*     | 80     | \*    | \*     | E2 82 AC    | 20 AC       |
| 中   | U+4E2D  | \*    | \*     | \*     | \*    | D6 D0  | E4 B8 AD    | 4E 2D       |
| 气   | U+6C14  | \*    | \*     | \*     | \*    | C6 F8  | E6 B0 94    | 6C 14       |
| 氣   | U+6C23  | \*    | \*     | \*     | \*    | \*     | E6 B0 A3    | 6C 23       |
| 𝄞    | U+1D11E | \*    | \*     | \*     | \*    | \*     | F0 9D 84 9E | D8 34 DD 1E |

可以看到：

1. ascii 字符的所有编码都相同(除了 utf-16 占了两个字节)
1. 除了 utf-8 和 utf-16, 其他编码都无法表示所有 Unicode 字符
1. 从`U+0000`到`U+FFFF`基本多文种平面, utf-16 编码即为 Unicode 编码，所有码字长度一致
1. 从`U+10000`到`U+10FFFF`辅助多文种平面, utf-16 编码也丧失了长度一致的特性

### Q&A

Q: utf-8 比 utf-16 的优势是什么？

A: 因为大多数文献的大部分内容都是英文字符，utf-8 处理这些字符兼容 ascii 编码，且只占用一个字符大小，所以 utf-8 编码是最常用的编码方式。

Q: 除了 utf-8 和 utf-16，其他编码方式还有存在的必要吗？

A: 有。基于兼容性考虑，很多现成的文档已经是 cp1252 编码，如果要处理这些文档，就需要使用 cp1252 编码。对于新建文档，其他编码意义不大。

### 编码异常

除了 utf-8 和 utf-16 以外，其他编码方式都无法处理所有 Unicode 字符，所以在编码时，如果遇到无法编码的字符，就会抛出异常。

```python
>>> '中'.encode('ascii')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)
```

解决方法是指定错误处理方式，比如忽略错误字符。

```python
>>> '中'.encode('ascii', errors='ignore')
b''
>>> '中'.encode('ascii', errors='replace')
b'?'
>>> '中'.encode('ascii', errors='xmlcharrefreplace')
b'&#20013;'
```

### 解码异常

解码的问题更严重，当遇到无法解码的字符，会抛出异常。如果不知道原文的编码方式，有可能解码出错误的字符。

```python
>>> octets = b'Montr\xe9al'
>>> octets.decode('cp1252')
'Montréal'
>>> octets.decode('iso8859_7')
'Montrιal'
>>> octets.decode('koi8_r')
'MontrИal'
>>> octets.decode('utf-8')
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe9 in position 5:
invalid continuation byte
>>> octets.decode('utf-8', errors='replace')
'MontrÝal'
```

### 小插曲：Python 源代码的编码

注意，Python 源代码 py 文件也是文本文件，也需要编码。Python3 默认使用 utf-8 编码，所以没有特殊的问题，但是在 Python2 中，你可能经常会看见这样的头部注释。

```python
# coding: cp1252
```

这是告诉 Python2 解析器，这个文件使用 cp1252 编码来解析源代码。这行代码必须写在头部，因为它可以用 ascii 编码规则来解码，在此之前，Python2 解析器并不知道文件的编码方式。

### 如何得到一个文件的编码方式？

在解码异常章节中，我们提到，如果不知道原文的编码方式，有可能解码出错误的字符。那么接下来的一个问题是，如何知道一个文件的编码方式呢？

很遗憾，答案是：**没有办法!**，一个文件的编码方式并不存在于这个文件的 meta data 中。我们常见的通信协议，例如 http 和 xml，他们的头部都会包含编码方式，而文件却没有。

那怎么办呢？

答案是：**猜**。本文不展开讨论这个细节，但是有一个库可以帮助你猜测一个文件的编码方式，[chardet](https://pypi.org/project/chardet)。

### 什么是 BOM(Byte Order Mark)？

utf-16 需要一个字节序标记，用来指定编码的字节顺序，从而让解码器知道如何解码。

```python
>>> 'e'.encode('utf-16')
b'\xff\xfee\x00'
>>> b'\xff\xfee\x00'.decode('utf-16')
'e'
>>> b'\xfe\xff\x00e'.decode('utf-16')
'e'
```

其中第 1，2 个字节就是 BOM，0xFFFE 表示小端序，0xFEFF 表示大端序。有了 BOM，即使不在编码方式中指定小端序或这大端序，也可以在默认大端序的机器上解码小端序；在默认小端序的机器上解码大端序。

## 系统的默认的编码方式

这段代码可以查看系统的默认编码方式。

```python
import locale
import sys
expressions = """
    locale.getpreferredencoding()
    my_file.encoding
    sys.stdout.encoding
    sys.stdin.encoding
    sys.stderr.encoding
    sys.getdefaultencoding()
    sys.getfilesystemencoding()
"""
my_file = open('dummy', 'w')
for expression in expressions.split():
    value = eval(expression)
    print(f'{expression:>30} -> {value!r}')
```

这是 Windows 10 PowerShell 的输出结果。

```
 locale.getpreferredencoding() -> 'cp936'
              my_file.encoding -> 'cp936'
           sys.stdout.encoding -> 'UTF-8'
            sys.stdin.encoding -> 'utf-8'
           sys.stderr.encoding -> 'UTF-8'
      sys.getdefaultencoding() -> 'utf-8'
   sys.getfilesystemencoding() -> 'utf-8'
```

## Unicode 的归一化比较

问题的提出：
张三写了一篇关于 cafe 的文章，用的是字符串`cafe`，李四也写了一篇关于 cafe 的文章，用的是字符串`ｃａｆｅ`（全角字符），然后王五想找到所有关于 cafe 的文章，搜索了`cafe`。这时候，如果只是用字符串匹配，那么张三的文章被找到了，但是李四的文章被漏掉了？看个例子

```python
>>> 'cafe' == 'ｃａｆｅ'
False
```

他们的 Unicode 值不一样

```python
>>> ord('c')
99
>>> ord('ｃ')
65347
```

这个时候，我们需要将字符串先进行归一化，然后再比较。

```
>>> from unicodedata import normalize
>>> normalize('NFKC', 'cafe') == normalize('NFKC', 'ｃａｆｅ')
True
```

本小节内容涉及编码细节和四种规范化方式（NFC, NFD, NFKC, NFKD），感兴趣的同学可以看这篇文章[ Unicode Equivalence](https://en.wikipedia.org/wiki/Unicode_equivalence)。

## Unicode 数据库

Unicode 标准提供了一整套数据库，它不仅提供了字符到名字的映射，还通过 metadata 提供了字符的属性。
例如，一个字符是否可打印，是否是字母，是否是十进制数字，或者是否是其他数字符号。这就是`str`方法`isalpha`，`isprintable`，`isdecimal`和`isnumeric`如何工作的。`str.casefold`也使用 Unicode 表中的信息。

最后，我们来读两段关于 Unicode 数据库的有趣的代码。

第一段，它可以查找到所有关于 cat 和 smiling 的 emoji。

```python
import sys
import unicodedata

START, END = ord(" "), sys.maxunicode + 1


def find(*query_words, start=START, end=END):
    query = {w.upper() for w in query_words}
    for code in range(start, end):
        char = chr(code)
        name = unicodedata.name(char, None)
        if name and query.issubset(name.split()):
            print(f"U+{code:04X}\t{char}\t{name}")


find("cat", "smiling")
```

输出是

```python
U+1F638	😸	GRINNING CAT FACE WITH SMILING EYES
U+1F63A	😺	SMILING CAT FACE WITH OPEN MOUTH
U+1F63B	😻	SMILING CAT FACE WITH HEART-SHAPED EYES
```

第二段，它是一个关于 metadata 的例子

```
import unicodedata
import re

re_digit = re.compile(r"\d")
sample = "1\xbc\xb2\u0969\u136b\u216b\u2466\u2480\u3285"
for char in sample:
    print(
        f"U+{ord(char):04x}",
        char.center(6),
        "re_dig" if re_digit.match(char) else "-",
        "isdig" if char.isdigit() else "-",
        "isnum" if char.isnumeric() else "-",
        f"{unicodedata.numeric(char):5.2f}",
        unicodedata.name(char),
        sep="\t",
    )
```

输出是

```python
U+0031,   1   , re_dig, isdig, isnum,  1.00, DIGIT ONE
U+00bc,   ¼   , ------, -----, isnum,  0.25, VULGAR FRACTION ONE QUARTER
U+00b2,   ²   , ------, isdig, isnum,  2.00, SUPERSCRIPT TWO
U+0969,   ३   , re_dig, isdig, isnum,  3.00, DEVANAGARI DIGIT THREE
U+136b,   ፫   , ------, isdig, isnum,  3.00, ETHIOPIC DIGIT THREE
U+216b,   Ⅻ  , ------, -----, isnum, 12.00, ROMAN NUMERAL TWELVE
U+2466,   ⑦   , ------, isdig, isnum,  7.00, CIRCLED DIGIT SEVEN
U+2480,   ⒀   , ------, -----, isnum, 13.00, PARENTHESIZED NUMBER THIRTEEN
U+3285,   ㊅   , ------, -----, isnum,  6.00, CIRCLED IDEOGRAPH SIX
```

## 参考文献

1. Fluent Python, Second Edition
1. https://docs.python.org/3/howto/unicode.html
