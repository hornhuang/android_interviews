## 问题

**Java 的 char 是两个字节, 如何存 UTF-8 的字符的?**

## 考什么

- 是否熟悉 Java char 和字符串. (初级)
- 是否了解字符的映射和储存细节. (中级)
- 是否能触类旁通, 横向对比其他语言. (高级)

## 剖析

先分析题目中两个名词:

1. char 是什么?
2. UTF-8 是什么?
3. 然后各占几个字节?
4. 和 Unicode 什么关系?

#### char 是什么

甲骨文官方对于原始数据类型 char 定义:

> char: The char data type is a single 16-bit Unicode character. It has a minimum value of '\u0000' (or 0) and a maximum value of '\uffff' (or 65,535 inclusive).

可以知道主要以下三点:

- char 类型是原始类型
- Unicode 字符 (额外补充, 采用 UTF-16 的形式)
- 16 bit 即 2 个字节表示一个字符

char 里面存的是什么:



```csharp
System.out.println(Integer.toHexString('庆'));
```

打印出来的 0x5e, 0x86, 它们 2 个就是 Unicode 的码点.

Java 中的 char 占用 2 个字节.

#### UTF-8 是什么

UTF-8 是一种 Unicode 的编码形式. 它可能占用 1~4 个字节. 细节可以参考[Unicode ASCII UTF-8 GBK关系](https://www.jianshu.com/p/1a39be00f5b8)

例如 `char test = '庆'` 是用 2 个字节进行表示的. 而它是通过 UTF-8 编码为 `e5ba86` 储存的, 是 3 个字节.

> Unicode 和 ASCII 是字符集, 而 UTF-8, UTF-16 等是一种编码. 是 unicode 的表现形式.

#### 如何存储字符

字符（人类认知） -> char (字符集) -> byte (计算机存储)

char 中其实是 unicode 的字符集, 然后在计算机储存中通过 UTF-8 进行编码储存. 但请注意, 实际上 Java 中使用的是 UTF-16! 具体参考[编码之JVM之外与之内](https://www.jianshu.com/p/6f4739923fc2)

> UTF-8与UTF-16的区别 -
>  UTF-8 最小的单位是 1 个字节, UTF-16 最小的单位是 2 个字节.
>  也就是说, 如果这个字符是 a，a 对应 65，在 UTF-8 中它实际上和 ASCII 码相同, 也就是7个比特就可以表示. 但在 UTF-16 中仍要2个字节的.

如果把字符换成中文:
 类似于“中”字，占1个char就可以了，也就是2个字节。同样是用UTF-16。

额外的, 注意字节序的问题



```csharp
byte[] bytes= "中".getBytes("utf-16");
System.out.println(bytes.length);

for(byte b : bytes) {
    System.out.print(Integer.toHexString(Byte.toUnsignedInt(b)));
}
```

结果是:



```undefined
4
fe ff 4e 2d
```

其中 `fe ff` 是字节序的标志. 不是代表的真正的内容, 而只是让读取这个数据的人知道字节序是什么.

## 触类旁通 Python 的字符串



```python
byteString = "I'm a byte string."

# 例如utf-8，字面量会用utf-8编码成字节存入字符串
# coding = utf-8
byteString = "中国"      # .py文件中存放的是 UTF-8 编码后的字节
unicodeString = u"中国"  # .py文件中存放的是 UCS2(~UTF-16) 编码后的字节
```

Python 是解释执行的, 源文件与执行时内存中字符串内容一致。
 Javac指定编码将字符串统一转为 MUTF-8 (调用这个命令的时候需要给个encoding的参数)



```python
// Java
byte[] byteString= "中".getBytes("utf-8"); 

// 对应 Python
byteString = "中国"

// Java
String unicodeString = "中国" 

// 对应 Python
unicodeString = u"中国"
```

unicodeString可以调用encode(),byteString得decode().

## 让人迷惑的字符串长度

**字符串长度不等于字符数!**



```csharp
// emoji 代表表情
String emoji= "emoji"
System.out.println(emoji.length());
```

我们会认为看到的是 1, 所以输出是 1, 但表情占用 2 个 char. 输出 2. 在 Java 中看到的字符串长度不一定是输出的字符串长度.

但在 Python 中



```python
String emoji= u"emoji"
print(len(emoji))
```

python >= 3.2, 结果为 1, 低版本为 2.
 所以有时候输入框有字符限制, 用户输入一个表情减 2 个字符, 这不合理.

Java 9并没有改变字符串长度和字符数不一致的情况。
 但是对拉丁(Latin)字符的存储空间做了优化. UTF-16 对于 a, b, c 等可以1个字节的还是需要两个字节来存. 会改用 byte 来存. 节省一般空间.
 字符串长度 ！= 字符数

## 题目结论

- Java char 不存 UTF-8 的字节，而是 UTF-16 的.
- Unicode 通用字符集占2个字节，例如“中”.
- Unicode 是扩展字符集需要用一对char来表示，例如“emoji”
- Unicode 是字符集, 不是编码.
- Java String 字符串长度不等于字符数.