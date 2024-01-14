## 字符串常见操作

### 1. 分割、合并子串

利用trim和正则表达式可以很好分割字符串

join可以合并子串
```
s = s.trim();
String[] words= s.split("\\s+");
return String.join(" ",words);
```

### 2. 修改字符串

虽然java中String类型是不可修改的，但是java提出了StringBuilder和StringBuffer作为“可修改的String”。

其中StringBuffer是多线程安全的，而StringBuilder是面向单线程的，所有StringBuilder更快一些，两者的API是相同的。



