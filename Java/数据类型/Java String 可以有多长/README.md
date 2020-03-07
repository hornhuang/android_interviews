# 1、考察什么

- 对字符串编码有深入了解（中级）
- 对字符串在内存中的 存储形式有深入了解（中级）
- 对 Java 虚拟机字节码有足够的了解（中级）
- 对 Java 虚拟机指令有一定认识（中级）



# 2、题目剖析

- 字符串多长是指字符数还是字节数？

- 字符串有几种存在形式？
- 字符串不同形式受到何种限制？



# 3、Java String

- 栈 - 

```String longString = "aaa......aa"```

- 堆 - 

```byte[] bytes = loadFromFile(new File("superLongText.txt"))
byte[] bytes = loadFromFile(new File("superLongText.txt"))

String superLongString = new String(bytes);
```



